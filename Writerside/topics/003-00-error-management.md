# Chapitre 4 : La Gestion des Erreurs - L'Essentiel

Jusqu'à présent, nous avons construit les routes et les ponts de notre API. Tout fonctionne à merveille tant que les
utilisateurs suivent le chemin balisé. Mais que se passe-t-il si quelqu'un se trompe d'adresse ? Ou essaie de construire
une voiture avec des pièces défectueuses ? Une bonne API ne doit pas simplement afficher un message "ERREUR" cryptique.
Elle doit guider l'utilisateur, comme un copilote calme et précis.

### Objectifs Pédagogiques

À la fin de cette partie, vous serez capable de :

- Comprendre les limites de la gestion d'erreurs par défaut de Spring Boot.
- Créer un DTO standard pour représenter les erreurs de manière cohérente.
- Implémenter un gestionnaire d'exceptions global avec `@RestControllerAdvice`.
- Gérer les exceptions personnalisées pour les erreurs métier (ex: ressource non trouvée).
- Personnaliser la réponse d'erreur pour les échecs de validation (`400 Bad Request`) afin de la rendre plus claire pour
  le client.

### Introduction : Le Copilote de Votre API

Imaginez que vous utilisez une application GPS. Vous entrez une destination qui n'existe pas. Que préférez-vous ?

1. L'application plante avec un message "Erreur #54B".
2. L'application affiche un message clair : "Désolé, cette adresse est introuvable. Veuillez vérifier votre saisie."

La réponse est évidente. Une bonne gestion des erreurs est un élément clé de l'expérience utilisateur (ou de
l'expérience développeur pour le consommateur de votre API). Elle transforme la frustration en information utile.

Actuellement, notre API laisse Spring Boot gérer les erreurs. C'est un bon début, mais les messages sont génériques.
Nous allons construire notre propre "tour de contrôle" pour intercepter toutes les exceptions et les transformer en
réponses HTTP claires, cohérentes et professionnelles.

### Centraliser la Gestion des Erreurs avec `@RestControllerAdvice`

Le secret pour ne pas répéter la logique de gestion d'erreurs dans chaque méthode de chaque contrôleur est d'utiliser un
**gestionnaire d'exceptions global**. Dans Spring, cela se fait avec une classe annotée `@RestControllerAdvice`.

Cette classe agira comme un filet de sécurité. Toute exception non attrapée par un contrôleur sera interceptée ici. Nous
pourrons alors décider quelle réponse HTTP renvoyer pour chaque type d'exception.

#### 1. Un Format d'Erreur Standard : Le DTO d'Erreur

Avant de gérer les erreurs, définissons à quoi ressemblera une réponse d'erreur. Créons un DTO pour cela.

<procedure title="Création de ErrorDto.java">
Créez ce DTO dans votre package `fr.formation.spring.bibliotech.api.dto`.

```java
// package fr.formation.spring.bibliotech.api.dto;

import lombok.Data;

import java.time.LocalDateTime;
import java.util.Map;

@Data
public class ErrorDto {
    private LocalDateTime timestamp;
    private int status;
    private String error;
    private String message;
    private String path;
    // Utilisé pour les erreurs de validation
    private Map<String, String> validationErrors;

    public ErrorDto(int status, String error, String message, String path) {
        this.timestamp = LocalDateTime.now();
        this.status = status;
        this.error = error;
        this.message = message;
        this.path = path;
    }
}
```

</procedure>

#### 2. L'Exception Personnalisée : `ResourceNotFoundException`

Pour gérer le cas où une ressource n'est pas trouvée, il est plus propre de lancer une exception métier spécifique
plutôt que de retourner `null` ou un `Optional` vide jusqu'au contrôleur.

<procedure title="Création de ResourceNotFoundException.java">
Créez un nouveau package `fr.formation.spring.bibliotech.exceptions` et placez-y cette classe.

```java
// package fr.formation.spring.bibliotech.exceptions;

import org.springframework.http.HttpStatus;
import org.springframework.web.bind.annotation.ResponseStatus;

// @ResponseStatus est un fallback. Si l'exception n'est pas gérée par
// un @ExceptionHandler, Spring retournera ce statut.
@ResponseStatus(HttpStatus.NOT_FOUND)
public class ResourceNotFoundException extends RuntimeException {

    public ResourceNotFoundException(String resourceName, Long id) {
        super(String.format("La ressource '%s' avec l'id '%d' n'a pas été trouvée.",
                resourceName, id));
    }
}
```

</procedure>

Maintenant, modifions notre `BookController` pour utiliser cette exception. C'est bien plus lisible !

```java
// Dans BookController.java, méthode getBookById

import fr.formation.spring.bibliotech.exceptions.ResourceNotFoundException;

//...

@GetMapping("/{id}")
public ResponseEntity<BookDto> getBookById(@PathVariable Long id) {
    Book book = this.bookRepository.findById(id)
            .orElseThrow(() -> new ResourceNotFoundException("Book", id));

    return ResponseEntity.ok(this.bookMapper.toDto(book));
}
```

#### 3. Le Gestionnaire Global : `RestExceptionHandler`

Voici la pièce maîtresse. Créez cette classe dans un nouveau package `fr.formation.spring.bibliotech.api.errors`.

<procedure title="Création de RestExceptionHandler.java">

```java
// package fr.formation.spring.bibliotech.api.errors;

import fr.formation.spring.bibliotech.api.dto.ErrorDto;
import fr.formation.spring.bibliotech.exceptions.ResourceNotFoundException;
import jakarta.servlet.http.HttpServletRequest;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.MethodArgumentNotValidException;
import org.springframework.web.bind.annotation.ExceptionHandler;
import org.springframework.web.bind.annotation.RestControllerAdvice;

import java.util.HashMap;
import java.util.Map;

@RestControllerAdvice
public class RestExceptionHandler {

    // Gère notre exception personnalisée pour les ressources non trouvées
    @ExceptionHandler(ResourceNotFoundException.class)
    public ResponseEntity<ErrorDto> handleResourceNotFound(
            ResourceNotFoundException ex, HttpServletRequest request) {

        ErrorDto errorDto = new ErrorDto(
                HttpStatus.NOT_FOUND.value(),
                "Resource Not Found",
                ex.getMessage(),
                request.getRequestURI());

        return new ResponseEntity<>(errorDto, HttpStatus.NOT_FOUND);
    }

    // Gère les erreurs de validation de @Valid
    @ExceptionHandler(MethodArgumentNotValidException.class)
    public ResponseEntity<ErrorDto> handleValidationExceptions(
            MethodArgumentNotValidException ex, HttpServletRequest request) {

        Map<String, String> errors = new HashMap<>();
        ex.getBindingResult().getFieldErrors().forEach(error ->
                errors.put(error.getField(), error.getDefaultMessage())
        );

        ErrorDto errorDto = new ErrorDto(
                HttpStatus.BAD_REQUEST.value(),
                "Validation Error",
                "Un ou plusieurs champs sont invalides.",
                request.getRequestURI());
        errorDto.setValidationErrors(errors);

        return new ResponseEntity<>(errorDto, HttpStatus.BAD_REQUEST);
    }
}
```

</procedure>

**Comment ça marche ?**

1. L'annotation `@RestControllerAdvice` scanne tous les contrôleurs de l'application.
2. Quand la méthode `getBookById` lance une `ResourceNotFoundException`, elle n'est pas attrapée localement.
3. Le `RestExceptionHandler` l'intercepte car il a une méthode annotée avec
   `@ExceptionHandler(ResourceNotFoundException.class)`.
4. Cette méthode s'exécute, construit notre `ErrorDto` standardisé et renvoie une réponse `404 Not Found` propre.

Désormais, si vous appelez `GET /api/books/999`, vous obtiendrez une réponse JSON claire et prévisible !

```json
{
  "timestamp": "2023-10-27T10:30:00.12345",
  "status": 404,
  "error": "Resource Not Found",
  "message": "La ressource 'Book' avec l'id '999' n'a pas été trouvée.",
  "path": "/api/books/999",
  "validationErrors": null
}
```

### Exercice 6 : Gérer les Conflits de Données

Un cas d'erreur métier très courant est la tentative de création d'une ressource qui viole une contrainte d'unicité (par
exemple, un email ou un ISBN déjà existant).

**Énoncé :**

1. Créez une nouvelle exception personnalisée `DuplicateResourceException.java` dans le package `exceptions`. Elle
   prendra en paramètre le nom de la ressource, le nom du champ et la valeur dupliquée.
2. Dans `BookRepository`, ajoutez une méthode pour vérifier l'existence d'un livre par son ISBN :
   `boolean existsByIsbn(String isbn);`.
3. Dans `BookController`, modifiez la méthode `createBook` pour vérifier si un livre avec le même ISBN existe déjà avant
   de le sauvegarder. Si c'est le cas, lancez votre nouvelle `DuplicateResourceException`.
4. Dans `RestExceptionHandler`, ajoutez une nouvelle méthode `handleDuplicateResource` pour attraper cette exception.
   Elle devra retourner une réponse avec le statut `409 CONFLICT` et un `ErrorDto` correctement rempli.
5. Testez en créant un livre, puis en essayant d'en créer un second avec le même ISBN.

### Correction exercice 6 {collapsible="true"}

**1. La nouvelle exception `DuplicateResourceException.java`**

```java
// package fr.formation.spring.bibliotech.exceptions;

import org.springframework.http.HttpStatus;
import org.springframework.web.bind.annotation.ResponseStatus;

@ResponseStatus(HttpStatus.CONFLICT)
public class DuplicateResourceException extends RuntimeException {

    public DuplicateResourceException(String resourceName,
                                      String fieldName,
                                      Object fieldValue) {
        super(String.format("La ressource '%s' avec %s = '%s' existe déjà.",
                resourceName, fieldName, fieldValue));
    }
}
```

**2. La méthode dans `BookRepository.java`**

```java
// package fr.formation.spring.bibliotech.dal.repositories;

import fr.formation.spring.bibliotech.dal.entities.Book;
import org.springframework.data.jpa.repository.JpaRepository;

public interface BookRepository extends JpaRepository<Book, Long> {
    // Nouvelle méthode pour vérifier l'unicité de l'ISBN
    boolean existsByIsbn(String isbn);
}
```

**3. La modification dans `BookController.java`**

```java
// Dans la méthode createBook du BookController

import fr.formation.spring.bibliotech.exceptions.DuplicateResourceException;

//...

@PostMapping
public ResponseEntity<BookDto> createBook(
        @Valid @RequestBody BookSaveDto bookToCreate) {

    // Ajout de la vérification d'unicité
    if (this.bookRepository.existsByIsbn(bookToCreate.getIsbn())) {
        throw new DuplicateResourceException("Book", "ISBN", bookToCreate.getIsbn());
    }

    Book newBook = this.bookMapper.toEntity(bookToCreate);
    Book savedBook = this.bookRepository.save(newBook);

    URI location = ServletUriComponentsBuilder
            .fromCurrentRequest().path("/{id}")
            .buildAndExpand(savedBook.getId()).toUri();

    return ResponseEntity.created(location)
            .body(this.bookMapper.toDto(savedBook));
}
```

**4. Le nouveau handler dans `RestExceptionHandler.java`**

```java
// Dans RestExceptionHandler.java

import fr.formation.spring.bibliotech.exceptions.DuplicateResourceException;

//...

@ExceptionHandler(DuplicateResourceException.class)
public ResponseEntity<ErrorDto> handleDuplicateResource(
        DuplicateResourceException ex, HttpServletRequest request) {

    ErrorDto errorDto = new ErrorDto(
            HttpStatus.CONFLICT.value(),
            "Duplicate Resource",
            ex.getMessage(),
            request.getRequestURI());

    return new ResponseEntity<>(errorDto, HttpStatus.CONFLICT);
}
```

**5. Test avec une requête HTTP**
D'abord, créez le livre de Tolkien.

```http
POST http://localhost:8080/api/books
Content-Type: application/json

{
  "title": "Le Seigneur des anneaux",
  "isbn": "978-2266120662",
  "publicationDate": "1954-07-29",
  "authorIds": [2]
}
```

Ensuite, essayez de le créer à nouveau avec le même ISBN.

```http
POST http://localhost:8080/api/books
Content-Type: application/json

{
  "title": "Un autre livre",
  "isbn": "978-2266120662",
  "publicationDate": "2024-01-01",
  "authorIds": [1]
}
```

Vous devriez recevoir une réponse `409 Conflict` avec un corps de message clair.

### Auto-évaluation

1. **(QCM)** Quelle annotation transforme une classe en un gestionnaire d'exceptions global pour les contrôleurs REST ?
   a) `@ControllerAdvice`
   b) `@ExceptionHandler`
   c) `@RestControllerAdvice`
   d) `@GlobalHandler`

2. **_ (Question ouverte)_** Quel est l'intérêt de créer une exception personnalisée comme `ResourceNotFoundException`
   au lieu de laisser une `NullPointerException` se propager ?

3. **(QCM)** Quel est le code de statut HTTP sémantiquement correct pour indiquer qu'une ressource ne peut pas être
   créée car elle existe déjà ?
   a) 400 Bad Request
   b) 404 Not Found
   c) 409 Conflict
   d) 500 Internal Server Error

4. **_ (Question ouverte)_** Dans un `@RestControllerAdvice`, comment peut-on accéder à l'URL de la requête qui a
   provoqué l'erreur ?

5. **_ (Question ouverte)_** Expliquez brièvement le rôle de la méthode `handleValidationExceptions` dans notre
   `RestExceptionHandler`. Que gère-t-elle et pourquoi est-ce utile ?

### Conclusion

Vous venez d'ajouter une couche de professionnalisme et de robustesse indispensable à votre API. En centralisant la
gestion des erreurs, vous avez rendu votre code de contrôleur plus propre et plus lisible. Plus important encore, votre
API communique désormais de manière claire et prévisible, même en cas de problème. Elle ne se contente plus de "tomber
en panne", elle dialogue avec son utilisateur.

Maintenant que notre API est fonctionnelle, propre et robuste, comment faire pour que d'autres développeurs puissent l'
utiliser facilement ? Comment leur expliquer quels endpoints sont disponibles, quels paramètres ils attendent et quelles
réponses ils retournent ? C'est le rôle de la documentation, et nous verrons dans le prochain chapitre comment
l'automatiser avec un outil puissant : Swagger/OpenAPI.