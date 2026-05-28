# Spring Boot et API REST

## Qu'est-ce que Spring Boot ?

**Spring Boot** est un framework Java qui simplifie la creation d'applications Spring en eliminant la configuration manuelle. Il est construit sur le **Spring Framework** (2003), l'un des ecosystemes Java les plus matures et utilises en enterprise.

> [!tip] Analogie
> Spring Boot est a Java ce que [[08 - APIs REST avec Flask|Flask]] / [[09 - APIs REST avec FastAPI|FastAPI]] sont a [[01 - Introduction a Python|Python]], mais avec beaucoup plus de batteries incluses. La ou FastAPI fournit le routage et la validation, Spring Boot fournit en plus : la gestion de la securite, l'ORM, la gestion des transactions, le monitoring, la documentation API automatique, et un serveur web embarque. C'est un ecosysteme complet, pas juste un framework web.

### Les trois piliers de Spring Boot

1. **Auto-configuration** : Spring Boot detecte les bibliotheques presentes dans le classpath et configure automatiquement les composants necessaires (datasource, serveur web, securite...).
2. **Starters** : dependances Maven/Gradle preconfiginees qui regroupent tout ce qu'il faut pour une fonctionnalite (`spring-boot-starter-web`, `spring-boot-starter-data-jpa`...).
3. **Serveur embarque** : Tomcat (ou Jetty/Undertow) est inclus dans le JAR — pas besoin de deployer sur un serveur externe.

---

## Creer un projet Spring Boot

### Via Spring Initializr

La facon la plus simple est [start.spring.io](https://start.spring.io) — un generateur de projet :

```
Project     : Maven
Language    : Java
Spring Boot : 3.2.x (derniere stable)
Group       : com.holberton
Artifact    : taskapi
Java        : 21

Dependances a cocher :
  - Spring Web
  - Spring Data JPA
  - H2 Database
  - Validation
  - Spring Security
  - Lombok (optionnel mais tres utile)
```

### Structure d'un projet Spring Boot

```
taskapi/
├── pom.xml
└── src/
    ├── main/
    │   ├── java/com/holberton/taskapi/
    │   │   ├── TaskapiApplication.java      # Point d'entree
    │   │   ├── controller/
    │   │   │   └── TaskController.java      # Couche HTTP
    │   │   ├── service/
    │   │   │   └── TaskService.java         # Logique metier
    │   │   ├── repository/
    │   │   │   └── TaskRepository.java      # Acces aux donnees
    │   │   ├── model/
    │   │   │   └── Task.java                # Entite JPA
    │   │   ├── dto/
    │   │   │   ├── TaskRequest.java         # DTO entrant
    │   │   │   └── TaskResponse.java        # DTO sortant
    │   │   └── exception/
    │   │       └── GlobalExceptionHandler.java
    │   └── resources/
    │       └── application.yml              # Configuration
    └── test/
        └── java/com/holberton/taskapi/
            ├── controller/
            │   └── TaskControllerTest.java
            └── service/
                └── TaskServiceTest.java
```

### pom.xml Spring Boot

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
         https://maven.apache.org/xsd/maven-4.0.0.xsd">
    
    <modelVersion>4.0.0</modelVersion>
    
    <!-- Spring Boot parent : gere les versions de TOUTES les dependances -->
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>3.2.4</version>
    </parent>
    
    <groupId>com.holberton</groupId>
    <artifactId>taskapi</artifactId>
    <version>1.0.0-SNAPSHOT</version>
    
    <properties>
        <java.version>21</java.version>
    </properties>
    
    <dependencies>
        <!-- Web MVC + serveur Tomcat embarque -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        
        <!-- JPA + Hibernate -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-jpa</artifactId>
        </dependency>
        
        <!-- Base H2 en memoire pour le dev -->
        <dependency>
            <groupId>com.h2database</groupId>
            <artifactId>h2</artifactId>
            <scope>runtime</scope>
        </dependency>
        
        <!-- Validation des inputs -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-validation</artifactId>
        </dependency>
        
        <!-- Securite -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-security</artifactId>
        </dependency>
        
        <!-- Tests -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>
    
    <build>
        <plugins>
            <!-- Plugin qui cree un JAR executable -->
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>
</project>
```

### Point d'entree de l'application

```java
package com.holberton.taskapi;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

// @SpringBootApplication = @Configuration + @EnableAutoConfiguration + @ComponentScan
@SpringBootApplication
public class TaskapiApplication {
    
    public static void main(String[] args) {
        SpringApplication.run(TaskapiApplication.class, args);
        // Spring Boot demarre Tomcat sur le port 8080 automatiquement
    }
}
```

```bash
# Lancer l'application
mvn spring-boot:run

# ou apres compilation
java -jar target/taskapi-1.0.0-SNAPSHOT.jar
```

---

## Configuration : application.yml

Spring Boot lit sa configuration depuis `application.properties` ou `application.yml`. Le format YAML est plus lisible pour les structures complexes :

```yaml
# src/main/resources/application.yml

spring:
  application:
    name: taskapi
  
  # Base de donnees H2 pour le developpement
  datasource:
    url: jdbc:h2:mem:taskdb;DB_CLOSE_DELAY=-1
    driver-class-name: org.h2.Driver
    username: sa
    password: ""
  
  # Console H2 (pour inspecter la BDD en dev)
  h2:
    console:
      enabled: true
      path: /h2-console
  
  # JPA / Hibernate (ORM Java — voir [[10 - Python et Bases de Donnees]] pour l'equivalent SQLAlchemy)
  jpa:
    hibernate:
      ddl-auto: create-drop  # cree le schema au demarrage, supprime a l'arret
    show-sql: true           # affiche les requetes SQL dans les logs
    properties:
      hibernate:
        format_sql: true

# Port du serveur
server:
  port: 8080

# Logging
logging:
  level:
    com.holberton: DEBUG
    org.springframework.security: INFO
```

---

## Architecture MVC : Controller / Service / Repository

Spring Boot encourage une architecture en couches clairement separees :

```
Client HTTP
    |
    v
Controller  ← reçoit la requete HTTP, delègue au Service
    |
    v
Service     ← logique metier, transactions, validations
    |
    v
Repository  ← acces aux donnees (JPA/SQL)
    |
    v
Base de donnees
```

> [!info] Injection de dependances (IoC)
> Spring gere lui-meme la creation et l'injection des objets (beans). Vous n'avez pas besoin d'instancier les services avec `new` — Spring le fait pour vous via l'**Inversion of Control (IoC)**. Declarez simplement vos dependances dans le constructeur avec `@Autowired` ou preferablement via l'injection par constructeur.

---

## Entite JPA : la couche Model

```java
package com.holberton.taskapi.model;

import jakarta.persistence.*;
import jakarta.validation.constraints.NotBlank;
import jakarta.validation.constraints.Size;
import java.time.LocalDateTime;

@Entity                           // Marque la classe comme une entite JPA (ORM — voir [[10 - Python et Bases de Donnees]] pour SQLAlchemy)
@Table(name = "tasks")            // Nom de la table [[01 - Introduction au SQL|SQL]] (optionnel, defaut = "task")
public class Task {
    
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    @NotBlank(message = "Le titre est obligatoire")
    @Size(min = 1, max = 255, message = "Le titre doit faire entre 1 et 255 caracteres")
    @Column(nullable = false)
    private String titre;
    
    @Size(max = 2000)
    @Column(columnDefinition = "TEXT")
    private String description;
    
    @Column(nullable = false)
    private boolean complete = false;
    
    @Column(name = "created_at", updatable = false)
    private LocalDateTime createdAt;
    
    @Column(name = "updated_at")
    private LocalDateTime updatedAt;
    
    // Callbacks JPA : appeles automatiquement avant persistence
    @PrePersist
    protected void onCreate() {
        createdAt = LocalDateTime.now();
        updatedAt = LocalDateTime.now();
    }
    
    @PreUpdate
    protected void onUpdate() {
        updatedAt = LocalDateTime.now();
    }
    
    // Constructeur sans argument requis par JPA
    protected Task() {}
    
    public Task(String titre, String description) {
        this.titre = titre;
        this.description = description;
    }
    
    // Getters et setters
    public Long getId() { return id; }
    public String getTitre() { return titre; }
    public void setTitre(String titre) { this.titre = titre; }
    public String getDescription() { return description; }
    public void setDescription(String description) { this.description = description; }
    public boolean isComplete() { return complete; }
    public void setComplete(boolean complete) { this.complete = complete; }
    public LocalDateTime getCreatedAt() { return createdAt; }
    public LocalDateTime getUpdatedAt() { return updatedAt; }
}
```

---

## Spring Data JPA : la couche Repository

```java
package com.holberton.taskapi.repository;

import com.holberton.taskapi.model.Task;
import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.data.jpa.repository.Query;
import org.springframework.data.repository.query.Param;
import org.springframework.stereotype.Repository;

import java.util.List;

// JpaRepository<Task, Long> : entite Task, type de la cle primaire Long
// Spring genere automatiquement l'implementation !
@Repository
public interface TaskRepository extends JpaRepository<Task, Long> {
    
    // Spring Data genere la requete SQL depuis le nom de la methode
    List<Task> findByComplete(boolean complete);
    List<Task> findByTitreContainingIgnoreCase(String motCle);
    List<Task> findByCompleteFalseOrderByCreatedAtDesc();
    
    // Requete JPQL (Java Persistence Query Language — similaire a [[01 - Introduction au SQL|SQL]] mais sur les entites)
    @Query("SELECT t FROM Task t WHERE t.titre LIKE %:motCle% OR t.description LIKE %:motCle%")
    List<Task> rechercherParMotCle(@Param("motCle") String motCle);
    
    // Requete SQL native
    @Query(value = "SELECT COUNT(*) FROM tasks WHERE complete = false", nativeQuery = true)
    long compterTachesEnAttente();
    
    // JpaRepository fournit deja : save(), findById(), findAll(), delete(), deleteById(), count()...
}
```

---

## DTOs : separer les couches

Les **DTOs (Data Transfer Objects)** separent la representation HTTP de l'entite JPA. C'est une pratique fondamentale : n'exposez jamais directement vos entites JPA en JSON. Pour la comparaison avec les schemas Pydantic en FastAPI, voir [[09 - APIs REST avec FastAPI]].

```java
package com.holberton.taskapi.dto;

import jakarta.validation.constraints.NotBlank;
import jakarta.validation.constraints.Size;

// DTO entrant : ce que le client envoie
public record TaskRequest(
    @NotBlank(message = "Le titre est obligatoire")
    @Size(min = 1, max = 255)
    String titre,
    
    @Size(max = 2000)
    String description
) {}

// DTO sortant : ce que l'API retourne (peut omettre des champs sensibles)
public record TaskResponse(
    Long id,
    String titre,
    String description,
    boolean complete,
    String createdAt
) {
    // Methode de fabrique : convertit une entite en DTO
    public static TaskResponse fromTask(Task task) {
        return new TaskResponse(
            task.getId(),
            task.getTitre(),
            task.getDescription(),
            task.isComplete(),
            task.getCreatedAt() != null ? task.getCreatedAt().toString() : null
        );
    }
}
```

> [!warning] Pourquoi ne pas exposer les entites JPA directement ?
> 1. **Couplage** : changer la BDD impacte l'API publique
> 2. **Securite** : les entites peuvent avoir des champs sensibles (password, token...)
> 3. **Cycles infinis** : les relations JPA (@OneToMany) peuvent causer des boucles JSON infinies
> 4. **Controle** : les DTOs permettent de formatter, renommer et filtrer les champs librement

---

## Controller REST

```java
package com.holberton.taskapi.controller;

import com.holberton.taskapi.dto.TaskRequest;
import com.holberton.taskapi.dto.TaskResponse;
import com.holberton.taskapi.service.TaskService;
import jakarta.validation.Valid;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;

import java.util.List;

// @RestController = @Controller + @ResponseBody
// Tous les retours sont serialises en JSON automatiquement
@RestController
@RequestMapping("/api/tasks")   // Prefixe commun a tous les endpoints
public class TaskController {
    
    private final TaskService taskService;
    
    // Injection par constructeur (recommande sur @Autowired sur le champ)
    public TaskController(TaskService taskService) {
        this.taskService = taskService;
    }
    
    // GET /api/tasks — lister toutes les taches
    @GetMapping
    public List<TaskResponse> getTasks(
            @RequestParam(required = false) Boolean complete,
            @RequestParam(required = false) String recherche) {
        return taskService.getTaches(complete, recherche);
    }
    
    // GET /api/tasks/42 — recuperer une tache par ID
    @GetMapping("/{id}")
    public ResponseEntity<TaskResponse> getTask(@PathVariable Long id) {
        return taskService.getTache(id)
            .map(ResponseEntity::ok)
            .orElse(ResponseEntity.notFound().build());
    }
    
    // POST /api/tasks — creer une tache
    @PostMapping
    @ResponseStatus(HttpStatus.CREATED)  // retourne 201 au lieu de 200
    public TaskResponse createTask(@Valid @RequestBody TaskRequest request) {
        return taskService.creerTache(request);
    }
    
    // PUT /api/tasks/42 — modifier une tache entierement
    @PutMapping("/{id}")
    public ResponseEntity<TaskResponse> updateTask(
            @PathVariable Long id,
            @Valid @RequestBody TaskRequest request) {
        return taskService.modifierTache(id, request)
            .map(ResponseEntity::ok)
            .orElse(ResponseEntity.notFound().build());
    }
    
    // PATCH /api/tasks/42/complete — marquer comme terminee
    @PatchMapping("/{id}/complete")
    public ResponseEntity<TaskResponse> completeTask(@PathVariable Long id) {
        return taskService.marquerComplete(id)
            .map(ResponseEntity::ok)
            .orElse(ResponseEntity.notFound().build());
    }
    
    // DELETE /api/tasks/42 — supprimer une tache
    @DeleteMapping("/{id}")
    @ResponseStatus(HttpStatus.NO_CONTENT)  // 204 : supprime avec succes
    public void deleteTask(@PathVariable Long id) {
        taskService.supprimerTache(id);
    }
}
```

---

## Service : la logique metier

```java
package com.holberton.taskapi.service;

import com.holberton.taskapi.dto.TaskRequest;
import com.holberton.taskapi.dto.TaskResponse;
import com.holberton.taskapi.model.Task;
import com.holberton.taskapi.repository.TaskRepository;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;

import java.util.List;
import java.util.Optional;
import java.util.stream.Collectors;

@Service
@Transactional  // Toutes les methodes s'executent dans une transaction par defaut
public class TaskService {
    
    private final TaskRepository taskRepository;
    
    public TaskService(TaskRepository taskRepository) {
        this.taskRepository = taskRepository;
    }
    
    @Transactional(readOnly = true)  // Optimisation pour les lectures
    public List<TaskResponse> getTaches(Boolean complete, String recherche) {
        List<Task> taches;
        
        if (recherche != null && !recherche.isBlank()) {
            taches = taskRepository.rechercherParMotCle(recherche);
        } else if (complete != null) {
            taches = taskRepository.findByComplete(complete);
        } else {
            taches = taskRepository.findAll();
        }
        
        return taches.stream()
            .map(TaskResponse::fromTask)
            .collect(Collectors.toList());
    }
    
    @Transactional(readOnly = true)
    public Optional<TaskResponse> getTache(Long id) {
        return taskRepository.findById(id)
            .map(TaskResponse::fromTask);
    }
    
    public TaskResponse creerTache(TaskRequest request) {
        Task task = new Task(request.titre(), request.description());
        Task saved = taskRepository.save(task);
        return TaskResponse.fromTask(saved);
    }
    
    public Optional<TaskResponse> modifierTache(Long id, TaskRequest request) {
        return taskRepository.findById(id).map(task -> {
            task.setTitre(request.titre());
            task.setDescription(request.description());
            return TaskResponse.fromTask(taskRepository.save(task));
        });
    }
    
    public Optional<TaskResponse> marquerComplete(Long id) {
        return taskRepository.findById(id).map(task -> {
            task.setComplete(!task.isComplete());  // toggle
            return TaskResponse.fromTask(taskRepository.save(task));
        });
    }
    
    public void supprimerTache(Long id) {
        if (!taskRepository.existsById(id)) {
            throw new TaskNotFoundException("Tache non trouvee : " + id);
        }
        taskRepository.deleteById(id);
    }
}
```

---

## Gestion globale des erreurs

```java
package com.holberton.taskapi.exception;

// Exception metier
public class TaskNotFoundException extends RuntimeException {
    public TaskNotFoundException(String message) {
        super(message);
    }
}

// DTO de reponse d'erreur
public record ErrorResponse(
    int status,
    String error,
    String message,
    String timestamp
) {}

// Gestionnaire global des exceptions HTTP
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.MethodArgumentNotValidException;
import org.springframework.web.bind.annotation.ExceptionHandler;
import org.springframework.web.bind.annotation.RestControllerAdvice;

import java.time.LocalDateTime;
import java.util.stream.Collectors;

@RestControllerAdvice  // s'applique a tous les @RestController
public class GlobalExceptionHandler {
    
    // Gere les erreurs de validation (@Valid)
    @ExceptionHandler(MethodArgumentNotValidException.class)
    public ResponseEntity<ErrorResponse> handleValidation(MethodArgumentNotValidException ex) {
        String erreurs = ex.getBindingResult().getFieldErrors().stream()
            .map(e -> e.getField() + " : " + e.getDefaultMessage())
            .collect(Collectors.joining(", "));
        
        ErrorResponse response = new ErrorResponse(
            HttpStatus.BAD_REQUEST.value(),
            "Validation echouee",
            erreurs,
            LocalDateTime.now().toString()
        );
        return ResponseEntity.badRequest().body(response);
    }
    
    // Gere les ressources non trouvees
    @ExceptionHandler(TaskNotFoundException.class)
    public ResponseEntity<ErrorResponse> handleNotFound(TaskNotFoundException ex) {
        ErrorResponse response = new ErrorResponse(
            HttpStatus.NOT_FOUND.value(),
            "Ressource introuvable",
            ex.getMessage(),
            LocalDateTime.now().toString()
        );
        return ResponseEntity.status(HttpStatus.NOT_FOUND).body(response);
    }
    
    // Filet de securite : toute autre exception non geree
    @ExceptionHandler(Exception.class)
    public ResponseEntity<ErrorResponse> handleGeneric(Exception ex) {
        ErrorResponse response = new ErrorResponse(
            HttpStatus.INTERNAL_SERVER_ERROR.value(),
            "Erreur interne",
            "Une erreur inattendue s'est produite",
            LocalDateTime.now().toString()
        );
        return ResponseEntity.internalServerError().body(response);
    }
}
```

---

## Spring Security : authentification JWT

```java
package com.holberton.taskapi.security;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.http.HttpMethod;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.config.annotation.web.configuration.EnableWebSecurity;
import org.springframework.security.config.http.SessionCreationPolicy;
import org.springframework.security.web.SecurityFilterChain;
import org.springframework.security.web.authentication.UsernamePasswordAuthenticationFilter;

@Configuration
@EnableWebSecurity
public class SecurityConfig {
    
    private final JwtFilter jwtFilter;
    
    public SecurityConfig(JwtFilter jwtFilter) {
        this.jwtFilter = jwtFilter;
    }
    
    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        return http
            // Desactiver CSRF (inutile pour les APIs stateless)
            .csrf(csrf -> csrf.disable())
            
            // Sans session : chaque requete porte son propre token
            .sessionManagement(session ->
                session.sessionCreationPolicy(SessionCreationPolicy.STATELESS))
            
            // Regles d'autorisation
            .authorizeHttpRequests(auth -> auth
                // Endpoints publics
                .requestMatchers("/api/auth/**").permitAll()
                .requestMatchers("/h2-console/**").permitAll()
                .requestMatchers(HttpMethod.GET, "/api/tasks").permitAll()
                // Tout le reste necessite une authentification
                .anyRequest().authenticated()
            )
            
            // Ajouter notre filtre JWT avant le filtre d'authentification standard
            .addFilterBefore(jwtFilter, UsernamePasswordAuthenticationFilter.class)
            
            .build();
    }
}

// Service JWT simplifie
import io.jsonwebtoken.Claims;
import io.jsonwebtoken.Jwts;
import io.jsonwebtoken.SignatureAlgorithm;
import org.springframework.stereotype.Component;

import java.util.Date;

@Component
public class JwtService {
    
    private static final String SECRET_KEY = "ma-cle-secrete-tres-longue-pour-hs256";
    private static final long EXPIRATION_MS = 86_400_000L; // 24 heures
    
    public String genererToken(String username) {
        return Jwts.builder()
            .setSubject(username)
            .setIssuedAt(new Date())
            .setExpiration(new Date(System.currentTimeMillis() + EXPIRATION_MS))
            .signWith(SignatureAlgorithm.HS256, SECRET_KEY)
            .compact();
    }
    
    public String extraireUsername(String token) {
        return extraireClaims(token).getSubject();
    }
    
    public boolean validerToken(String token) {
        try {
            Claims claims = extraireClaims(token);
            return !claims.getExpiration().before(new Date());
        } catch (Exception e) {
            return false;
        }
    }
    
    private Claims extraireClaims(String token) {
        return Jwts.parser()
            .setSigningKey(SECRET_KEY)
            .parseClaimsJws(token)
            .getBody();
    }
}
```

---

## Documentation API avec SpringDoc (Swagger)

```xml
<!-- Ajout dans pom.xml -->
<dependency>
    <groupId>org.springdoc</groupId>
    <artifactId>springdoc-openapi-starter-webmvc-ui</artifactId>
    <version>2.3.0</version>
</dependency>
```

```yaml
# application.yml
springdoc:
  api-docs:
    path: /api-docs
  swagger-ui:
    path: /swagger-ui.html
```

```java
// Annotation des endpoints pour la documentation
import io.swagger.v3.oas.annotations.Operation;
import io.swagger.v3.oas.annotations.Parameter;
import io.swagger.v3.oas.annotations.responses.ApiResponse;
import io.swagger.v3.oas.annotations.tags.Tag;

@RestController
@RequestMapping("/api/tasks")
@Tag(name = "Tasks", description = "Gestion des taches")
public class TaskController {
    
    @GetMapping("/{id}")
    @Operation(
        summary = "Recuperer une tache par son ID",
        responses = {
            @ApiResponse(responseCode = "200", description = "Tache trouvee"),
            @ApiResponse(responseCode = "404", description = "Tache non trouvee")
        }
    )
    public ResponseEntity<TaskResponse> getTask(
            @Parameter(description = "ID de la tache") @PathVariable Long id) {
        // ...
    }
}
```

Apres demarrage, la documentation interactive est accessible sur `http://localhost:8080/swagger-ui.html`.

---

## Tests avec JUnit 5 et Mockito

### Tests unitaires du Service (voir [[01 - Tests Unitaires et TDD]] pour les principes TDD et l'analogie JUnit/pytest)

```java
package com.holberton.taskapi.service;

import com.holberton.taskapi.dto.TaskRequest;
import com.holberton.taskapi.model.Task;
import com.holberton.taskapi.repository.TaskRepository;
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;
import org.junit.jupiter.api.extension.ExtendWith;
import org.mockito.InjectMocks;
import org.mockito.Mock;
import org.mockito.junit.jupiter.MockitoExtension;

import java.util.Optional;

import static org.assertj.core.api.Assertions.*;
import static org.mockito.ArgumentMatchers.any;
import static org.mockito.Mockito.*;

@ExtendWith(MockitoExtension.class)  // Active Mockito sans Spring
class TaskServiceTest {
    
    @Mock
    private TaskRepository taskRepository;  // Mock : version simulee du repository
    
    @InjectMocks
    private TaskService taskService;  // Classe testee avec le mock injecte
    
    private Task tacheExemple;
    
    @BeforeEach
    void setup() {
        tacheExemple = new Task("Apprendre Spring Boot", "Faire le cours Holberton");
    }
    
    @Test
    void creerTache_ShouldReturnTaskResponse_WhenValidRequest() {
        // GIVEN
        TaskRequest request = new TaskRequest("Nouvelle tache", "Description");
        Task saved = new Task(request.titre(), request.description());
        when(taskRepository.save(any(Task.class))).thenReturn(saved);
        
        // WHEN
        var response = taskService.creerTache(request);
        
        // THEN
        assertThat(response.titre()).isEqualTo("Nouvelle tache");
        assertThat(response.complete()).isFalse();
        verify(taskRepository, times(1)).save(any(Task.class));
    }
    
    @Test
    void getTache_ShouldReturnEmpty_WhenTaskNotFound() {
        // GIVEN
        when(taskRepository.findById(999L)).thenReturn(Optional.empty());
        
        // WHEN
        var result = taskService.getTache(999L);
        
        // THEN
        assertThat(result).isEmpty();
    }
    
    @Test
    void supprimerTache_ShouldThrow_WhenTaskNotFound() {
        // GIVEN
        when(taskRepository.existsById(999L)).thenReturn(false);
        
        // WHEN / THEN
        assertThatThrownBy(() -> taskService.supprimerTache(999L))
            .isInstanceOf(TaskNotFoundException.class)
            .hasMessageContaining("999");
    }
}
```

### Tests d'integration du Controller

```java
package com.holberton.taskapi.controller;

import com.fasterxml.jackson.databind.ObjectMapper;
import com.holberton.taskapi.dto.TaskRequest;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.autoconfigure.web.servlet.WebMvcTest;
import org.springframework.boot.test.mock.mockito.MockBean;
import org.springframework.http.MediaType;
import org.springframework.test.web.servlet.MockMvc;

import static org.mockito.ArgumentMatchers.any;
import static org.mockito.Mockito.when;
import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.*;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.*;

// @WebMvcTest : charge uniquement la couche Controller (pas la BDD)
@WebMvcTest(TaskController.class)
class TaskControllerTest {
    
    @Autowired
    private MockMvc mockMvc;  // Client HTTP de test
    
    @Autowired
    private ObjectMapper objectMapper;  // Serialisation JSON
    
    @MockBean
    private TaskService taskService;  // Mock du service
    
    @Test
    void createTask_ShouldReturn201_WhenValidRequest() throws Exception {
        TaskRequest request = new TaskRequest("Nouvelle tache", "Description");
        TaskResponse response = new TaskResponse(1L, "Nouvelle tache", "Description", false, null);
        
        when(taskService.creerTache(any())).thenReturn(response);
        
        mockMvc.perform(post("/api/tasks")
                .contentType(MediaType.APPLICATION_JSON)
                .content(objectMapper.writeValueAsString(request)))
            .andExpect(status().isCreated())
            .andExpect(jsonPath("$.titre").value("Nouvelle tache"))
            .andExpect(jsonPath("$.complete").value(false));
    }
    
    @Test
    void createTask_ShouldReturn400_WhenTitreBlank() throws Exception {
        TaskRequest request = new TaskRequest("", "Description");
        
        mockMvc.perform(post("/api/tasks")
                .contentType(MediaType.APPLICATION_JSON)
                .content(objectMapper.writeValueAsString(request)))
            .andExpect(status().isBadRequest())
            .andExpect(jsonPath("$.error").value("Validation echouee"));
    }
}

// Test d'integration complet (charge tout Spring)
@SpringBootTest
@AutoConfigureMockMvc
class TaskApiIntegrationTest {
    
    @Autowired
    private MockMvc mockMvc;
    
    @Test
    void fullCrudFlow() throws Exception {
        // Creer
        String createBody = """
            {"titre": "Ma tache", "description": "Details de la tache"}
            """;
        String createResponse = mockMvc.perform(post("/api/tasks")
                .contentType(MediaType.APPLICATION_JSON)
                .content(createBody))
            .andExpect(status().isCreated())
            .andReturn().getResponse().getContentAsString();
        
        // Extraire l'ID et tester les autres operations...
    }
}
```

---

## Relation One-to-Many avec JPA

```java
// Entite User (un utilisateur possede plusieurs taches)
@Entity
@Table(name = "users")
public class User {
    
    @Id @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    private String username;
    
    // @OneToMany : un User peut avoir plusieurs Task
    // mappedBy = nom du champ dans Task qui pointe vers User
    // CascadeType.ALL : les operations se propagent aux Task
    // orphanRemoval : supprime les Task si elles sont retirees de la liste
    @OneToMany(mappedBy = "user", cascade = CascadeType.ALL, orphanRemoval = true)
    private List<Task> tasks = new ArrayList<>();
    
    // Methodes helper pour maintenir la coherence des deux cotes de la relation
    public void addTask(Task task) {
        tasks.add(task);
        task.setUser(this);
    }
    
    public void removeTask(Task task) {
        tasks.remove(task);
        task.setUser(null);
    }
}

// Cote Task : le cote "Many"
@Entity
public class Task {
    // ...
    
    @ManyToOne(fetch = FetchType.LAZY)  // LAZY : ne charge pas l'user par defaut
    @JoinColumn(name = "user_id")       // Nom de la cle etrangere dans la table
    private User user;
}
```

> [!warning] N+1 Problem — le piegeon JPA classique
> Si vous chargez 100 Task avec une relation `@ManyToOne` en LAZY, et que votre code accede a `task.getUser()` pour chacune, JPA execute 100 requetes supplementaires (1 par tache). Solution : utiliser `@EntityGraph` ou une requete JPQL avec `JOIN FETCH` pour charger les donnees en une seule requete.

---

## Exercices pratiques

> [!tip] Conseil d'environnement
> Utilisez Spring Initializr pour generer le projet, puis importez dans IntelliJ IDEA ou VS Code avec l'extension Spring Boot.

### Exercice 1 — API Livres

Creer une API REST complete `GET/POST/PUT/DELETE /api/livres` avec :
- Entite `Livre` (id, titre, auteur, isbn, anneePublication, disponible)
- Repository avec methode de recherche par auteur et par titre (contient)
- Service avec logique metier : `emprunter(id)` et `retourner(id)` qui verifient la disponibilite
- Controller avec validation Bean sur tous les champs
- Gestion d'erreur globale : `LivreNotFoundException`, `LivreIndisponibleException`

### Exercice 2 — Relations JPA

Etendre l'API Livres avec :
- Entite `Utilisateur` (id, nom, email) avec relation `@OneToMany` vers ses emprunts
- Entite `Emprunt` (id, livre, utilisateur, dateEmprunt, dateRetour)
- Endpoint `POST /api/emprunts` qui cree un emprunt (verifie disponibilite + limite 3 livres par user)
- Endpoint `GET /api/utilisateurs/{id}/emprunts` pour voir les emprunts actifs d'un utilisateur

### Exercice 3 — Tests complets (voir [[01 - Tests Unitaires et TDD]])

Pour l'API Taches du cours :
- Ecrire 5 tests unitaires du `TaskService` avec Mockito (cas normaux + cas d'erreur)
- Ecrire 5 tests `@WebMvcTest` du `TaskController` (validations, codes HTTP, corps JSON)
- Ecrire 2 tests d'integration `@SpringBootTest` (flow creation -> lecture -> suppression)

### Exercice 4 — Pagination et tri

Modifier l'endpoint `GET /api/tasks` pour supporter :
- `?page=0&size=10` (pagination via `Pageable` Spring Data)
- `?sort=createdAt,desc` (tri dynamique)
- Retourner un objet `Page<TaskResponse>` avec les metadonnees (totalElements, totalPages, currentPage)

### Exercice 5 — Profils Spring

Configurer deux profils (`dev` et `prod`) :
- `dev` : H2 en memoire, logs DEBUG, h2-console active, securite desactivee
- `prod` : [[01 - Introduction au SQL|PostgreSQL]], logs INFO uniquement, h2-console desactivee, JWT obligatoire
- Fichiers : `application.yml`, `application-dev.yml`, `application-prod.yml`
- Lancer avec : `mvn spring-boot:run -Dspring-boot.run.profiles=dev`

---

## Notes liées

- [[01 - Introduction a Java]] — prerequis : syntaxe, types, Maven
- [[02 - Java POO et Collections]] — prerequis : POO, interfaces, generics, Streams
- [[08 - APIs REST avec Flask]] — API REST Python avec Flask a comparer avec Spring Boot
- [[09 - APIs REST avec FastAPI]] — API REST Python avec FastAPI, schemas Pydantic vs DTOs Java
- [[10 - Python et Bases de Donnees]] — ORM SQLAlchemy Python vs Spring Data JPA
- [[01 - Introduction au SQL]] — SQL, requetes, relations — base pour JPQL et les entites JPA
- [[01 - Tests Unitaires et TDD]] — TDD, JUnit 5, Mockito — principes et pratique
- [[01 - Docker]] — conteneuriser une application Spring Boot
- [[04 - CI-CD avec GitHub Actions]] — pipeline CI/CD pour un projet Spring Boot Maven
