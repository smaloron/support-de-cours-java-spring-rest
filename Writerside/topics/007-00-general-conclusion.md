# Conclusion générale

## Ce que vous avez appris

Reprenons un instant le fil de notre aventure pour voir tout ce que vous avez appris à maîtriser :

1. **Les Fondations (Chapitre 1)** : Vous avez appris à mettre en place un projet Spring Boot solide, à le connecter à
   une base de données et à le peupler avec des données initiales. Vous avez posé les fondations de notre "BiblioTech".

2. **Les Principes de REST (Chapitre 2)** : Vous avez découvert le "code de la route" du web. Vous comprenez maintenant
   la philosophie REST, le rôle des ressources, des URIs et des verbes HTTP. Vous avez construit vos premiers endpoints
   pour réaliser des opérations CRUD complètes.

3. **La Qualité des Données (Chapitre 3)** : Vous êtes passé à un niveau supérieur de professionnalisme en découplant
   votre API de votre base de données grâce aux DTOs. Vous avez appris à protéger votre application avec la validation
   des données et à automatiser le mapping avec MapStruct, gagnant en sécurité et en productivité.

4. **La Robustesse (Chapitre 4)** : Vous savez maintenant comment gérer les imprévus. En centralisant la gestion des
   erreurs, vous avez rendu votre API plus fiable et plus facile à utiliser, transformant les erreurs en dialogue
   informatif plutôt qu'en impasse.

5. **La Communication (Chapitre 5)** : Vous avez compris qu'une bonne API est une API bien documentée. Vous êtes capable
   de générer une documentation interactive et toujours à jour avec OpenAPI et Swagger, rendant votre travail accessible
   et facile à intégrer pour d'autres développeurs.

6. **La Finesse (Chapitre 6)** : Vous avez exploré les territoires au-delà du CRUD. Vous savez maintenant modéliser des
   recherches complexes, des relations hiérarchiques et des actions métier spécifiques, ce qui vous permet de concevoir
   des API qui collent parfaitement à la réalité de n'importe quel domaine.

Enfin, vous avez consolidé ces savoirs avec des **conseils et bonnes pratiques** qui feront de vous non seulement un bon
technicien, mais un véritable artisan du logiciel, soucieux de la qualité et de la clarté de son travail.

## L'Architecte, c'est Vous !

Au début de ce cours, je vous ai présenté la métaphore de l'architecte qui construit des ponts entre les applications.
Aujourd'hui, cet architecte, c'est vous. Vous avez les compétences pour concevoir et construire ces ponts de
communication qui sont au cœur de la quasi-totalité des applications modernes. Que ce soit pour une application mobile,
une application web, un système d'objets connectés ou une architecture de microservices, les principes que vous avez
appris restent les mêmes.

## Et maintenant ? Pistes pour aller plus loin

Votre voyage de formation ne s'arrête pas là. Voici quelques pistes pour approfondir encore vos compétences :

- **La Sécurité en profondeur** : Nous avons effleuré la sécurité. Plongez dans **Spring Security** pour apprendre à
  mettre en place une authentification et une autorisation complètes (JWT, OAuth2, gestion des rôles et des
  permissions).
- **Les Tests d'Intégration** : Comment s'assurer que vos endpoints fonctionnent comme prévu après chaque modification ?
  Apprenez à écrire des tests d'intégration avec des outils comme `RestAssured` ou `MockMvc` de Spring pour automatiser
  la vérification de votre API.
- **Les Architectures Asynchrones** : Toutes les communications n'ont pas besoin d'être synchrones. Explorez les files
  de messages (comme RabbitMQ ou Kafka) et l'API `CompletableFuture` de Java pour construire des systèmes plus réactifs
  et résilients.
- **GraphQL** : REST n'est pas la seule manière de construire des API. Découvrez GraphQL, une alternative populaire
  proposée par Facebook, qui offre une approche différente, particulièrement puissante pour les applications front-end
  complexes.
- **Les Microservices** : Les compétences que vous avez acquises sont le prérequis indispensable pour aborder le monde
  des microservices. Apprenez comment plusieurs petites API peuvent collaborer pour former une grande application, en
  utilisant des outils comme Spring Cloud (Gateway, Service Discovery, etc.).

Le plus important est de **continuer à pratiquer**. Prenez une idée d'application qui vous plaît (un gestionnaire de
recettes, un suivi de vos séries préférées, un petit outil pour votre club de sport...) et construisez son API de A à Z
en appliquant tout ce que vous avez appris. Il n'y a pas de meilleur moyen de consolider ses connaissances que de les
mettre en œuvre sur un projet personnel.

Je vous souhaite une excellente continuation dans votre carrière de Concepteur Développeur d'Application. Vous avez des
bases solides, soyez curieux, continuez d'apprendre, et construisez de grandes choses !

---

## Corrections des Auto-évaluations {collapsible="true"}

### Chapitre 1

1. (b) Gérer les dépendances du projet et sa construction.
2. L'annotation `@Entity` indique à JPA que cette classe Java correspond à une table dans la base de données et que ses
   instances peuvent être persistées.
3. (b) `h2`.
4. L'interface `CommandLineRunner` permet d'exécuter du code au démarrage de l'application, une fois le contexte Spring
   chargé, ce qui est idéal pour initialiser la base avec des données de test.
5. `@ManyToMany` indique une relation où plusieurs instances d'une entité peuvent être associées à plusieurs instances
   d'une autre. Elle est utilisée entre `Book` et `Author` car un livre peut avoir plusieurs auteurs et un auteur peut
   écrire plusieurs livres.

### Chapitre 2 (L'Essentiel)

1. (b) Representational State Transfer.
2. Une ressource est une entité (donnée ou objet) suffisamment importante pour être identifiée et manipulée via une API.
   Dans "BiblioTech", un livre (`Book`), un auteur (`Author`) ou une bibliothèque (`Library`) sont des ressources.
3. (c) `@GetMapping`.
4. Le principe "Stateless" signifie que le serveur ne stocke aucune information sur l'état du client entre deux
   requêtes. C'est un avantage car cela simplifie le serveur et facilite grandement la montée en charge (scalabilité).
5. En suivant les conventions, l'URI de base serait `/api/libraries`.

### Chapitre 2 (Pour aller plus loin)

1. (c) `@PathVariable`.
2. `POST` est utilisé pour créer une nouvelle ressource (il n'est pas idempotent). `PUT` est utilisé pour remplacer
   intégralement une ressource existante à une URI connue (il est idempotent).
3. (b) `201 Created`.
4. `ResponseEntity` permet de contrôler finement la réponse HTTP : (1) on peut définir n'importe quel code de statut (
   200, 201, 404, etc.) et (2) on peut ajouter des en-têtes HTTP personnalisés (comme `Location` après une création).
5. Elle devrait retourner un code de statut `404 Not Found` pour indiquer que la ressource demandée n'existe pas.

### Chapitre 3 (L'Essentiel)

1. (c) Découpler le contrat d'API de la structure de la base de données.
2. Car les informations nécessaires pour créer une ressource (ex: `authorIds`) sont souvent différentes des informations
   que l'on veut afficher après sa création (ex: `authorNames`, `id` généré). Cela améliore la sécurité et la clarté.
3. (c) Le Mapper.
4. (1) Ils préviennent l'exposition de données sensibles ou internes des entités. (2) Ils résolvent les problèmes de
   sérialisation en boucle infinie causés par les relations bidirectionnelles.
5. Le repository est nécessaire pour transformer les identifiants d'auteurs (`Set<Long>`) fournis dans le `BookSaveDto`
   en véritables entités `Author` en allant les chercher dans la base de données.

### Chapitre 3 (Pour aller plus loin)

1. (c) `@Valid`.
2. Car MapStruct est un processeur d'annotations qui génère du code Java pur à la compilation. Il n'utilise pas la
   réflexion à l'exécution, ce qui le rend aussi rapide que du code écrit à la main.
3. (b) Sur les DTOs d'entrée.
4. Il indique à MapStruct de générer une implémentation du mapper qui est un bean Spring (`@Component`), la rendant
   ainsi injectable dans d'autres composants comme les contrôleurs.
5. `400 Bad Request`.

### Chapitre 4

1. (c) `@RestControllerAdvice`.
2. Cela permet de créer une erreur sémantique, spécifique au domaine métier. C'est plus clair et permet de la gérer de
   manière ciblée dans un gestionnaire d'exceptions, plutôt que de traiter une erreur technique générique.
3. (c) `409 Conflict`.
4. En ajoutant un paramètre de type `HttpServletRequest` à la méthode du `@ExceptionHandler`. On peut alors appeler
   `request.getRequestURI()`.
5. Elle intercepte les exceptions de type `MethodArgumentNotValidException`, levées lorsque la validation via `@Valid`
   échoue. C'est utile pour transformer le message d'erreur par défaut de Spring en un format JSON personnalisé et plus
   lisible pour le client.

### Chapitre 5 (L'Essentiel)

1. (c) `springdoc-openapi-starter-webmvc-ui`.
2. `http://localhost:8080/swagger-ui.html`.
3. (d) `@Operation`.
4. Elle permet de fournir une description et des exemples pour un champ spécifique d'un DTO, ce qui clarifie sa
   signification pour les utilisateurs de l'API dans la documentation Swagger.
5. Les annotations de contrôleur (`@RestController`, `@RequestMapping`), les annotations de méthode (`@GetMapping`,
   `@PostMapping`, etc.), les paramètres de méthode (`@PathVariable`, `@RequestBody`) et les types de retour.

### Chapitre 5 (Pour aller plus loin)

1. (b) En créant un `Bean` de type `OpenAPI` dans une classe de configuration.
2. L'objet `Pageable` de `org.springframework.data.domain`.
3. (b) Définir un `SecurityScheme` dans la configuration `OpenAPI` et utiliser `@SecurityRequirement` sur la méthode.
4. L'annotation `@Parameter` sert à documenter un paramètre d'une méthode de contrôleur (paramètre de chemin, de
   requête...). Par exemple : `@Parameter(description = "ID de l'utilisateur", example = "123")`.
5. Le bouton "Authorize".

### Chapitre 6

1. (c) Utiliser les Spécifications Spring Data JPA.
2. Par exemple, `GET /api/libraries/{libraryId}/books` pour lister tous les livres disponibles dans une bibliothèque
   spécifique.
3. (c) `POST /api/commands/{id}/validation`. C'est une action qui n'est pas idempotente, et l'URI représente bien l'
   action comme une sous-ressource.
4. Les Spécifications permettent de construire dynamiquement une requête complexe à partir de multiples critères
   optionnels, sans avoir à créer une méthode pour chaque combinaison possible de critères dans le Repository. C'est
   beaucoup plus flexible et évolutif.
5. `Specification.where(null)` crée une spécification neutre qui ne filtre rien. C'est un point de départ pratique sur
   lequel on peut ensuite chaîner des critères avec `.and()` de manière conditionnelle.
   {/collapsible}

### Chapitre 6 Pagination et Tri

1. **(c) `Pageable`.** C'est l'interface de `org.springframework.data.domain` qui encapsule les informations de
   pagination et de tri.
2. (1) Pour éviter de surcharger le réseau et le serveur en transférant des volumes de données massifs. (2) Pour
   améliorer l'expérience utilisateur côté client, en chargeant les données progressivement.
3. **(b) Dans le champ `content`.**
4. `?page=2&size=10` (les pages sont indexées à partir de 0).
5. On peut passer plusieurs paramètres `sort` : `?sort=firstName,asc&sort=lastName,desc`.

Absolument ! Il semble qu'il y ait eu une petite réorganisation des chapitres au fil de nos échanges. Je vais vous
fournir la correction pour la section qui était spécifiquement dédiée à **HATEOAS**, qui était initialement une partie
avancée du chapitre 6.

Voici la correction pour l'auto-évaluation sur **HATEOAS**.

---

### Chapitre 6 HATEOAS

1. **(b) Hypermedia As The Engine Of Application State.** C'est la définition complète de l'acronyme. L'idée centrale
   est que l'état de l'application cliente est "piloté" par les liens (hypermédia) que le serveur lui envoie.

2. Le principal avantage pour le client est le **découplage fort** avec le serveur. Le client n'a plus besoin de
   connaître à l'avance et de coder en dur la structure des URIs de l'API. Il lui suffit de connaître le point d'entrée,
   puis de suivre les liens fournis dans les réponses. Si le serveur change ses URIs, le client continue de fonctionner
   sans modification tant qu'il suit les relations (`rel`) qu'il connaît.

3. **(c) `RepresentationModel`.** Dans les versions récentes de Spring HATEOAS, un DTO doit hériter de
   `RepresentationModel<T>` pour pouvoir y attacher des liens. (Note : les plus anciennes versions utilisaient une
   classe nommée `ResourceSupport`).

4. La construction `linkTo(methodOn(...))` est un moyen **type-safe** et **refactor-safe** de créer des liens.
    - **Type-safe** : Le compilateur vérifie que la méthode que vous appelez existe bien sur le contrôleur.
    - **Refactor-safe** : Si vous renommez l'URI de l'endpoint dans son annotation `@RequestMapping`, ou si vous
      renommez la méthode Java elle-même, le lien généré se mettra à jour automatiquement. Cela évite les liens cassés
      qui se produiraient si vous construisiez l'URL manuellement avec des chaînes de caractères.

5. La relation "self" est une convention standard qui fournit l'**URI canonique** de la ressource elle-même. C'est en
   quelque sorte le lien "vous êtes ici" ou l'adresse unique et permanente de la ressource qui est représentée dans la
   réponse.

### Chapitre 6 Le Versioning d'API

1. **(b) Pour gérer les "breaking changes" sans impacter les clients existants.** C'est l'objectif fondamental du
   versioning : permettre à l'API d'évoluer tout en garantissant la stabilité pour les applications qui en dépendent.

2. Un "breaking change" (ou changement cassant) est une modification apportée à l'API qui rompt le contrat avec les
   clients existants, les obligeant à modifier leur code pour continuer à fonctionner.
    - **Exemple concret** : Dans notre `BookDto`, si nous décidions que le champ `isbn` (une `String`) devait devenir un
      objet complexe pour mieux représenter ses différentes parties, comme
      `{"type": "ISBN-13", "value": "978-0439708180"}`, ce serait un "breaking change". Les clients qui s'attendaient à
      recevoir une simple chaîne de caractères pour le champ `isbn` planteraient en essayant de désérialiser la nouvelle
      structure.

3. **(b) Ajouter un nouveau champ optionnel `editor` à `BookDto`.** C'est un changement non-cassant, car les
   clients existants ignoreront simplement ce nouveau champ qu'ils ne connaissent pas. Le reste de la structure de la
   réponse reste le même, donc leur code de désérialisation continuera de fonctionner. Les autres options sont toutes
   des "breaking changes".

4. La meilleure pratique est de **ne pas modifier le contrôleur de la V1**. On crée un **nouveau fichier de contrôleur
   ** (par exemple, `BookControllerV2.java`), on l'annote avec le nouveau chemin de version (
   `@RequestMapping("/api/v2/books")`), et on y implémente la nouvelle logique. Ainsi, les deux versions coexistent et
   sont servies simultanément par l'application.

5. **Stratégie de versioning par URI** :
    - **Avantage :** C'est la stratégie la plus **claire et la plus explicite**. L'URL est facile à lire, à partager, à
      tester dans un navigateur et à mettre en cache par les serveurs intermédiaires.
    - **Inconvénient :** Certains puristes de REST argumentent que l'URI identifie une ressource, et que la ressource
      elle-même ne change pas de version. L'URI devrait donc rester stable (`/api/books`) et la version devrait être
      communiquée par un autre moyen (comme un en-tête `Accept`).
