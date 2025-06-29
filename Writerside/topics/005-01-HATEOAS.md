# Chapitre 5 : L'API Auto-découvrable avec HATEOAS - Pour aller plus loin

Nous avons construit une API puissante et bien structurée. Un client qui a lu notre documentation Swagger sait
exactement comment naviguer : pour voir les auteurs d'un livre, il sait qu'il doit construire l'URL
`/api/books/{id}/authors`. Mais que se passerait-il si nous pouvions rendre notre API si intelligente qu'elle dirait
elle-même au client quelles sont les prochaines étapes possibles ? C'est la promesse de HATEOAS.

### Objectifs Pédagogiques

À la fin de cette partie, vous serez capable de :

- Définir le principe HATEOAS (Hypermedia as the Engine of Application State).
- Comprendre les avantages d'une API auto-découvrable en termes de couplage faible.
- Intégrer la bibliothèque `spring-hateoas` dans votre projet.
- Modifier vos DTOs et contrôleurs pour inclure des liens hypermédia dans les réponses JSON.
- Créer des liens vers des ressources, des collections et des actions.

### Introduction : La Carte au Trésor Intelligente

Imaginez que votre API est une carte au trésor. Jusqu'à présent, vous avez fourni une carte complète (votre
documentation Swagger) au début de l'aventure. L'explorateur (le client de l'API) doit constamment la consulter pour
savoir où aller. S'il est sur l'île "Livre 1", il doit regarder la carte pour savoir que le "Village des Auteurs" se
trouve aux coordonnées X, Y.

**HATEOAS** (Hypermedia as the Engine of Application State) change la donne. C'est une carte au trésor qui parle. Quand
l'explorateur arrive sur l'île "Livre 1", il trouve une pancarte qui dit :

- "Vous êtes sur l'île Livre 1 (lien vers soi-même)".
- "Pour voir les auteurs de ce livre, suivez ce chemin : `/api/books/1/authors`".
- "Pour archiver ce livre, utilisez ce levier : `POST /api/books/1/archive`".

Le client n'a plus besoin de connaître à l'avance la structure des URIs. Il lui suffit de connaître le point d'entrée et
de suivre les liens fournis par l'API. L'API pilote l'état de l'application cliente à travers des hypermédias (des
liens).

### 1. Intégration de Spring HATEOAS

Comme toujours avec Spring, l'intégration est très simple. Il suffit d'ajouter une dépendance à notre `pom.xml`.

<procedure title="Ajout de la dépendance Spring HATEOAS">

```xml
<!-- Dans la section <dependencies> de votre pom.xml -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-hateoas</artifactId>
</dependency>
```

Cette dépendance va enrichir notre projet avec des outils pour construire des liens et formater les réponses JSON selon
la norme HAL (Hypertext Application Language).

</procedure>

### 2. Rendre nos DTOs "conscients des liens"

Pour qu'un DTO puisse transporter des liens, il doit hériter de la classe `RepresentationModel`.

<procedure title="Mise à jour de BookDto">

```java
// package fr.formation.spring.bibliotech.api.dto;

import org.springframework.hateoas.RepresentationModel;
// ...

// On fait hériter notre DTO de RepresentationModel<BookDto>
public class BookDto extends RepresentationModel<BookDto> {
    // ... les autres champs ne changent pas
    private Long id;
    private String title;
    // ...
}
```

Faites de même pour `AuthorDto`. `BookSaveDto`, lui, n'a pas besoin de liens car il est utilisé en entrée.

</procedure>

### 3. Ajouter des Liens dans le Contrôleur

Maintenant, nous devons construire et ajouter ces liens dans nos méthodes de contrôleur. Spring HATEOAS fournit une API
fluide et élégante pour cela avec les méthodes statiques `linkTo` et `methodOn`.

<procedure title="Refactorisation de BookController avec HATEOAS">
Modifions `getBookById` et `getAllBooks` (version paginée).

```java
// Dans BookController.java

import static org.springframework.hateoas.server.mvc.WebMvcLinkBuilder.*;

// ...
public class BookController {
    // ...

    @GetMapping("/{id}")
    public ResponseEntity<BookDto> getBookById(@PathVariable Long id) {
        Book book = this.bookRepository.findById(id)
                .orElseThrow(() -> new ResourceNotFoundException("Book", id));

        BookDto bookDto = this.bookMapper.toDto(book);

        // Ajout des liens HATEOAS
        // 1. Lien vers la ressource elle-même (self link)
        bookDto.add(linkTo(methodOn(BookController.class).getBookById(id)).withSelfRel());

        // 2. Lien vers la collection de tous les livres
        bookDto.add(linkTo(methodOn(BookController.class).searchBooks(null, null, null, null))
                .withRel("books"));

        // 3. Lien vers les auteurs de ce livre (sous-ressource)
        bookDto.add(linkTo(methodOn(BookController.class).getAuthorsOfBook(id))
                .withRel("authors"));

        return ResponseEntity.ok(bookDto);
    }
}
```

</procedure>

**Comment ça marche ?**

- `linkTo(methodOn(Controller.class).methode(...))` : Cette construction magique génère une URI pointant vers la méthode
  spécifiée. C'est refactor-safe : si vous renommez l'URI dans le `@GetMapping`, le lien se mettra à jour
  automatiquement !
- `.withSelfRel()` : Définit la relation du lien comme étant "self" (norme IANA).
- `.withRel("books")` : Définit une relation personnalisée. Le client saura que ce lien mène à la collection des livres.

Maintenant, une requête `GET /api/books/1` ne retournera plus seulement les données, mais aussi les actions possibles :

```json
{
  "id": 1,
  "title": "Harry Potter and the Sorcerer's Stone",
  "isbn": "978-0439708180",
  "publicationDate": "1997-06-26",
  "authorNames": [
    "J.K. Rowling"
  ],
  "_links": {
    "self": {
      "href": "http://localhost:8080/api/books/1"
    },
    "books": {
      "href": "http://localhost:8080/api/books/search"
    },
    "authors": {
      "href": "http://localhost:8080/api/books/1/authors"
    }
  }
}
```

<warning title="Le découplage ultime">
<p>
Le client n'a plus besoin de connaître la structure <code>/api/books/{id}/authors</code>. Il reçoit un objet <code>Book</code>, voit qu'il y a un lien avec la relation (<code>rel</code>) "authors", et il n'a plus qu'à suivre l'URL fournie dans le <code>href</code>. Si demain vous décidez de changer l'URI en <code>/api/v1/livres/{id}/ecrivains</code>, le client continuera de fonctionner sans aucune modification, car il ne fait que suivre les liens qu'on lui donne !
</p>
</warning>

### Exercice 10 : Rendre les Auteurs "découvrables"

C'est à votre tour d'appliquer le principe HATEOAS à notre `AuthorController`.

**Énoncé :**

1. Assurez-vous que votre `AuthorDto` hérite de `RepresentationModel<AuthorDto>`.
2. Modifiez la méthode `getAuthorById` dans `AuthorController` pour qu'elle ajoute les liens suivants à la réponse
   `AuthorDto` :
    - Un lien "self" pointant vers l'auteur lui-même.
    - Un lien "authors" pointant vers la collection de tous les auteurs (`getAllAuthors`).
    - Un lien "books" pointant vers la sous-ressource listant les livres de cet auteur.
3. Testez l'endpoint `GET /api/authors/1` et vérifiez que le corps JSON de la réponse contient bien le bloc `_links`avec
   les bonnes URLs.

### Correction exercice 10 {collapsible="true"}

**1. `AuthorDto.java` modifié**

```java
// package fr.formation.spring.bibliotech.api.dto;

import lombok.Data;
import org.springframework.hateoas.RepresentationModel;

@Data
@Schema(description = "Représentation d'un auteur après création.")
public class AuthorDto extends RepresentationModel<AuthorDto> {
    // ... les champs restent les mêmes
}
```

**2. `AuthorController.java` modifié**

```java
// package fr.formation.spring.bibliotech.api;

import static org.springframework.hateoas.server.mvc.WebMvcLinkBuilder.*;
//...

@RestController
@RequestMapping("/api/authors")
public class AuthorController {
    // ...

    @GetMapping("/{id}")
    public ResponseEntity<AuthorDto> getAuthorById(@PathVariable Long id) {
        return this.authorRepository.findById(id)
                .map(author -> {
                    AuthorDto authorDto = this.authorMapper.toDto(author);

                    // Ajout des liens HATEOAS
                    authorDto.add(linkTo(methodOn(AuthorController.class)
                            .getAuthorById(id)).withSelfRel());

                    authorDto.add(linkTo(methodOn(AuthorController.class)
                            .getAllAuthors(null)).withRel("authors"));

                    authorDto.add(linkTo(methodOn(AuthorController.class)
                            .getBooksByAuthor(id)).withRel("books"));

                    return authorDto;
                })
                .map(ResponseEntity::ok)
                .orElse(ResponseEntity.notFound().build());
    }
    // ...
}
```

**3. Résultat JSON attendu pour `GET /api/authors/1`**

```json
{
  "id": 1,
  "firstName": "J.K.",
  "lastName": "Rowling",
  "_links": {
    "self": {
      "href": "http://localhost:8080/api/authors/1"
    },
    "authors": {
      "href": "http://localhost:8080/api/authors"
    },
    "books": {
      "href": "http://localhost:8080/api/authors/1/books"
    }
  }
}
```

### Auto-évaluation

1. **(QCM)** Que signifie l'acronyme HATEOAS ?
   a) Hypertext And Text Engine Of Application State
   b) Hypermedia As The Engine Of Application State
   c) High-level Application Text Object And State
   d) Hypermedia And Timed-out Events On Application State

2. **_ (Question ouverte)_** Quel est le principal avantage de l'utilisation de HATEOAS pour le client d'une API ?

3. **(QCM)** Quelle classe de base de Spring HATEOAS un DTO doit-il étendre pour pouvoir contenir des liens ?
   a) `LinkedModel`
   b) `ResourceSupport`
   c) `RepresentationModel`
   d) `HypermediaModel`

4. **_ (Question ouverte)_** Dans l'API de Spring HATEOAS, à quoi sert la construction `linkTo(methodOn(...))` ?

5. **_ (Question ouverte)_** Dans une réponse HATEOAS, que représente la relation (`rel`) "self" ?

### Conclusion

Vous avez atteint le sommet de la montagne REST. HATEOAS est le principe qui distingue une "bonne" API d'une API
véritablement "RESTful". En rendant vos réponses auto-descriptives, vous créez des API incroyablement résilientes au
changement et faciles à explorer pour les clients. C'est un concept puissant qui, bien que demandant un peu plus de
travail côté serveur, offre des avantages considérables sur le long terme en termes de maintenabilité et de découplage.

Vous disposez maintenant d'un panorama complet des techniques, des patterns et des philosophies qui animent le
développement d'API REST modernes avec Spring Boot. Vous êtes prêt à concevoir et à construire des services web de haute
qualité.