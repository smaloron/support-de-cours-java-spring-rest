# Conseils et Bonnes Pratiques

Félicitations ! Vous avez parcouru un long chemin et maîtrisez maintenant les outils et les concepts pour construire des
API REST robustes avec Spring Boot. Mais comme pour tout artisan, connaître ses outils n'est que la moitié du travail.
L'autre moitié est de savoir quand et comment les utiliser avec sagesse et discernement. Cette section est une boîte à
outils de conseils et de bonnes pratiques issus de l'expérience du terrain.

### Objectifs Pédagogiques

À la fin de cette partie, vous serez capable de :

- Identifier et appliquer les conventions de nommage REST pour les URIs.
- Structurer votre projet de manière logique et évolutive.
- Comprendre l'importance de la gestion des versions de votre API.
- Appliquer des principes pour écrire un code de contrôleur propre et concis.
- Reconnaître les anti-patterns courants et savoir comment les éviter.

### Introduction : Le Guide du Bon Artisan

Vous êtes maintenant un constructeur de ponts (d'API). Vous savez comment poser les fondations (le projet Spring),
ériger les piliers (les contrôleurs), construire le tablier (les endpoints CRUD) et même ajouter des finitions comme les
garde-fous (gestion des erreurs) et la signalisation (documentation).

Mais comment s'assurer que votre pont est non seulement solide, mais aussi élégant, facile à traverser et simple à
entretenir ? C'est une question de métier, de respect des conventions et de bonnes habitudes. Ces conseils sont là pour
vous aider à passer de "constructeur fonctionnel" à "architecte d'API respecté".

### 1. Conventions de Nommage des URIs

La clarté de vos URIs est la première chose que les développeurs verront. Des URIs bien conçues sont auto-descriptives.

- **Utilisez des noms (substantifs), pas des verbes.**
    - :thumbsup: **Bien :** `GET /api/books`
    - :thumbsdown: **Mal :** `GET /api/getAllBooks`
      Le verbe est dans la méthode HTTP (GET, POST, ...). L'URI identifie la ressource.

- **Utilisez le pluriel pour les collections.**
    - :thumbsup: **Bien :** `/api/books`, `/api/authors`
    - :thumbsdown: **Mal :** `/api/book`, `/api/author`
      C'est intuitif : `/books` est la collection de tous les livres. `/books/1` est un livre spécifique au sein de
      cette collection.

- **Utilisez des minuscules et des tirets (`-`) pour la lisibilité.**
    - :thumbsup: **Bien :** `/api/best-sellers`
    - :thumbsdown: **Mal :** `/api/bestSellers` ou `/api/Best_Sellers`
      Les tirets sont préférés aux underscores ou au camelCase pour des raisons historiques et de compatibilité.

- **Préfixez vos URIs avec une version.**
    - :thumbsup: **Bien :** `/api/v1/books`
    - :thumbsdown: **Mal :** `/api/books`
      Nous y reviendrons, mais versionner votre API dès le début est une pratique salutaire.

### 2. Structure du Projet

Une structure de package claire rend votre projet plus facile à naviguer et à maintenir. La structure que nous avons
utilisée est un bon point de départ, basée sur les couches techniques.

```
fr.formation.spring.bibliotech
├── api
│   ├── dto
│   │   ├── BookDto.java
│   │   └── BookSaveDto.java
│   ├── errors
│   │   └── RestExceptionHandler.java
│   ├── mapper
│   │   └── BookMapper.java
│   └── BookController.java
├── config
│   ├── DataSeeder.java
│   └── OpenApiConfig.java
├── dal
│   ├── entities
│   │   └── Book.java
│   ├── repositories
│   │   └── BookRepository.java
│   └── spec
│       └── BookSpecification.java
└── exceptions
    └── ResourceNotFoundException.java
```

<note title="Alternative : Structurer par fonctionnalité (Feature)">
<p>
Pour les très grands projets, une structure par fonctionnalité peut être plus adaptée. Chaque fonctionnalité métier (livres, auteurs, emprunts) a son propre package contenant ses contrôleurs, DTOs, services, etc.
</p>
<pre>
fr.formation.spring.bibliotech
├── book
│   ├── BookController.java
│   ├── BookDto.java
│   ├── Book.java
│   └── BookRepository.java
├── author
│   ├── AuthorController.java
│   └── ...
</pre>
Cette approche améliore la cohésion, mais peut être excessive pour des projets de taille moyenne.

</note>

### 3. Gestion des Versions (Versioning)

Votre API va évoluer. Vous allez ajouter des champs, en supprimer, changer des logiques. Ces changements peuvent "
casser" les applications clientes qui utilisent une version plus ancienne de votre API. Il est donc crucial de
versionner votre API.

La méthode la plus courante et la plus claire est **l'inclusion de la version dans l'URI**.

**URI :** `/api/v1/books`, `/api/v2/books`

**Comment l'implémenter ?**
Le plus simple est d'ajouter le préfixe de version dans l'annotation `@RequestMapping` de vos contrôleurs.

```java

@RestController
@RequestMapping("/api/v1/books")
public class BookControllerV1 {
    // ...
}
```

Quand vous devez introduire un changement non rétro-compatible (un "breaking change"), vous ne modifiez pas le
`BookControllerV1`. Vous créez une copie, `BookControllerV2`, que vous mappez sur `/api/v2/books`, et vous y appliquez
vos modifications. Les anciens clients peuvent continuer à utiliser la V1, tandis que les nouveaux clients peuvent
utiliser la V2.

### 4. Gardez vos Contrôleurs "maigres" (Skinny Controllers)

Le rôle d'un contrôleur est d'être un **chef d'orchestre**, pas de jouer de tous les instruments. Il doit :

1. Recevoir la requête HTTP.
2. Valider les données d'entrée.
3. Appeler la couche de service pour exécuter la logique métier.
4. Mapper le résultat en DTO.
5. Construire et renvoyer la réponse HTTP.

**Toute la logique métier complexe ne doit PAS être dans le contrôleur.** Elle doit être déléguée à une couche de
service (`@Service`).

<procedure title="Exemple avec une couche Service">

1. **Créez un `BookService.java`**

```java
// package fr.formation.spring.bibliotech.services;

import org.springframework.stereotype.Service;
//...

@Service
public class BookService {
    private final BookRepository bookRepository;

    // Constructeur...

    public Book archiveBook(Long id) {
        Book book = bookRepository.findById(id)
                .orElseThrow(() -> new ResourceNotFoundException("Book", id));

        if (book.isArchived()) {
            throw new IllegalStateException("Le livre est déjà archivé.");
        }
        // ... autre logique métier complexe ...

        book.setArchived(true);
        return bookRepository.save(book);
    }
}
```

2. **Simplifiez le contrôleur**

```java
// Dans BookController.java
// ...
public class BookController {
    private final BookService bookService; // On injecte le service
    // ...

    @PostMapping("/{id}/archive")
    public ResponseEntity<Void> archiveBook(@PathVariable Long id) {
        this.bookService.archiveBook(id); // On délègue !
        return ResponseEntity.noContent().build();
    }
}
```

Le contrôleur est maintenant bien plus simple et focalisé sur son rôle de gestionnaire de requêtes HTTP.

</procedure>

### 5. Anti-Patterns à Éviter

- **Le "Fat DTO" :** Utiliser le même DTO pour la lecture, la création et la mise à jour. Nous avons déjà vu pourquoi
  c'est une mauvaise idée. Séparez les DTOs par cas d'usage.

- **Ignorer les Codes de Statut HTTP :** Toujours renvoyer `200 OK`, même en cas d'erreur, avec un champ
  `{"success": false}` dans le JSON. C'est un anti-pattern majeur. Utilisez toute la richesse des codes de statut (201,
  204, 400, 404, 409, etc.) pour communiquer l'état de l'opération.

- **Exposer les IDs de la base de données partout :** Pour les ressources publiques, il peut être préférable d'utiliser
  un identifiant public (comme un UUID) dans l'API, différent de la clé primaire auto-incrémentée de la base. Cela évite
  de donner des informations sur le nombre d'enregistrements et prévient les attaques par énumération.

- **Réponses "bavardes" :** N'envoyez que les données dont le client a besoin pour un cas d'usage donné. Par exemple,
  une liste de livres (`GET /api/books`) n'a peut-être pas besoin de tous les détails de chaque livre. Vous pourriez
  avoir un `BookSummaryDto` pour les listes et un `BookDetailDto` pour la vue détaillée.

### Conclusion

Ces conseils ne sont pas des règles absolues, mais des guides éprouvés qui vous aideront à écrire des API REST
meilleures, plus propres et plus maintenables. La clé est la **cohérence** et **l'empathie** envers le développeur qui
utilisera votre API. Pensez à lui : rendez-lui la vie facile avec des URIs claires, des réponses prévisibles et une
structure logique.

En appliquant ces principes, vous ne créerez pas seulement des API qui fonctionnent, mais des API que les gens aimeront
utiliser. C'est la marque d'un véritable professionnel. Dans la section suivante, nous verrons quelques exemples
d'applications concrètes où ces concepts prennent tout leur sens.