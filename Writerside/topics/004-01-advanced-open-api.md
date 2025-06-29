# Chapitre 6 : Configuration Avancée d'OpenAPI - Pour aller plus loin

Nous avons une documentation fonctionnelle et descriptive, mais elle ressemble encore à la documentation par défaut.
Comment y ajouter l'identité de notre projet "BiblioTech" ? Comment organiser nos endpoints en plusieurs groupes ? Et
comment gérer des cas plus complexes comme l'upload de fichiers ou la sécurité ? C'est ce que nous allons voir dans
cette partie avancée.

### Objectifs Pédagogiques

À la fin de cette partie, vous serez capable de :

- Configurer globalement les métadonnées de l'API (titre, version, description, informations de contact).
- Organiser les endpoints en groupes logiques avec les `@Tag`.
- Gérer la documentation pour des paramètres de requête plus complexes comme la pagination.
- Définir les schémas de sécurité (par exemple, JWT ou OAuth2) pour que Swagger UI puisse envoyer des requêtes
  authentifiées.
- Comprendre comment documenter des types de contenu différents (par exemple, pour l'upload de fichiers).

### Introduction : Personnaliser le Mode d'Emploi

Pour reprendre notre métaphore, nous avons un mode d'emploi standard pour notre appareil. Maintenant, il est temps d'y
apposer le logo de notre marque, d'y ajouter une section "Dépannage avancé" et de spécifier les précautions de sécurité.
Une documentation d'API professionnelle ne se contente pas de décrire les endpoints, elle reflète l'identité du projet
et fournit toutes les informations contextuelles nécessaires à son utilisation.

Nous allons transformer notre documentation Swagger UI de base en un portail d'API sur mesure pour "BiblioTech".

### 1. Configuration Globale de l'API

Plutôt que de laisser le titre générique, nous pouvons définir toutes les informations de haut niveau de notre API en
créant un `Bean` de configuration.

<procedure title="Création d'un Bean de configuration OpenAPI">
Créez une classe de configuration (par exemple, `OpenApiConfig.java`) dans votre package `config`.

```java
// package fr.formation.spring.bibliotech.config;

import io.swagger.v3.oas.models.OpenAPI;
import io.swagger.v3.oas.models.info.Contact;
import io.swagger.v3.oas.models.info.Info;
import io.swagger.v3.oas.models.info.License;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class OpenApiConfig {

    @Bean
    public OpenAPI customOpenAPI() {
        return new OpenAPI()
                .info(new Info()
                        .title("API BiblioTech")
                        .version("1.0.0")
                        .description("Une API REST pour la gestion d'un système de bibliothèques. " +
                                "Cette API permet de gérer les livres, les auteurs et les bibliothèques.")
                        .termsOfService("http://swagger.io/terms/")
                        .contact(new Contact()
                                .name("Support BiblioTech")
                                .url("https://www.mon-entreprise.fr/support")
                                .email("support@bibliotech.com"))
                        .license(new License()
                                .name("Apache 2.0")
                                .url("http://springdoc.org")));
    }
}
```

</procedure>

Relancez l'application. La page Swagger UI affiche maintenant un en-tête personnalisé avec le titre, la version et
toutes les informations de contact que vous avez définies. C'est bien plus professionnel !

### 2. Documenter la Pagination

Dans une vraie application, retourner des milliers de livres d'un coup n'est pas une bonne idée. On utilise la *
*pagination**. Implémentons-la rapidement et voyons comment la documenter.

<procedure title="Ajout de la pagination au `BookController`">
Spring Data JPA facilite grandement la pagination avec l'objet `Pageable`.

```java
// Dans BookController.java

import org.springframework.data.domain.Page;
import org.springframework.data.domain.Pageable;

// ...

@Operation(summary = "Récupère une liste paginée de livres")
@GetMapping
public Page<BookDto> getAllBooks(Pageable pageable) {
    // La méthode findAll(pageable) de Spring Data retourne directement
    // un objet Page, qui contient la liste des éléments pour la page
    // demandée ainsi que les informations de pagination (nombre total
    // de pages, nombre total d'éléments, etc.).
    return this.bookRepository.findAll(pageable)
            .map(this.bookMapper::toDto);
}
```

`springdoc-openapi` est intelligent. Il va automatiquement détecter que la méthode prend un `Pageable` en paramètre et
ajoutera les paramètres de requête correspondants dans la documentation Swagger :

- `page` (le numéro de la page, commence à 0)
- `size` (le nombre d'éléments par page)
- `sort` (un critère de tri, ex: `title,asc`)

Vous pouvez tester cela directement dans Swagger UI en remplissant ces champs. Par exemple, pour obtenir la deuxième
page de 5 livres triés par titre : `GET /api/books?page=1&size=5&sort=title,desc`.

</procedure>

### 3. Documenter la Sécurité de l'API

Pour l'instant, notre API est ouverte à tous. Dans une application réelle, la plupart des endpoints seraient protégés.
Supposons que nous utilisions l'authentification par **Bearer Token** (typiquement avec JWT). Nous devons indiquer à
Swagger comment inclure ce token dans les requêtes.

<procedure title="Configuration du schéma de sécurité JWT">
Nous allons modifier notre `OpenApiConfig` pour déclarer un schéma de sécurité.

```java
// Dans OpenApiConfig.java

import io.swagger.v3.oas.models.security.SecurityRequirement;
import io.swagger.v3.oas.models.security.SecurityScheme;
import io.swagger.v3.oas.models.Components;

// ...

@Bean
public OpenAPI customOpenAPI() {
    final String securitySchemeName = "bearerAuth";
    return new OpenAPI()
            // Ajout de la configuration de sécurité
            .addSecurityItem(new SecurityRequirement().addList(securitySchemeName))
            .components(
                    new Components()
                            .addSecuritySchemes(securitySchemeName,
                                    new SecurityScheme()
                                            .name(securitySchemeName)
                                            .type(SecurityScheme.Type.HTTP)
                                            .scheme("bearer")
                                            .bearerFormat("JWT")
                            )
            )
            // Ajout des informations générales (déjà fait)
            .info(new Info()
                    //...
            );
}
```

</procedure>

Après avoir rechargé, un bouton "Authorize" apparaît en haut à droite de l'interface Swagger UI. Un développeur peut
cliquer dessus, coller son token JWT, et toutes les requêtes qu'il enverra depuis l'interface incluront automatiquement
l'en-tête `Authorization: Bearer <token>`.

<note class="alert alert-info" title="Et si seuls certains endpoints sont protégés ?">
<p>
La configuration ci-dessus applique la sécurité à <b>toute l'API</b>. Si vous voulez la spécifier par endpoint, vous pouvez retirer le <code>.addSecurityItem(...)</code> de la configuration globale et l'ajouter via une annotation sur les méthodes ou les classes de contrôleur qui le nécessitent : <code>@SecurityRequirement(name = "bearerAuth")</code>.
</p>
</note>

### Exercice 8 : Améliorer la Documentation des Auteurs

Appliquons ces concepts avancés à notre API des auteurs.

**Énoncé :**

1. Dans `AuthorController`, modifiez la méthode `getAllAuthors` pour qu'elle accepte un `Pageable` et retourne une
   `Page<AuthorDto>`.
2. Dans `AuthorController`, imaginez que la suppression d'un auteur est une opération sensible réservée aux
   administrateurs. Ajoutez une annotation `@SecurityRequirement` sur la méthode `deleteAuthor` pour indiquer qu'elle
   nécessite l'authentification "bearerAuth" (que nous avons définie globalement).
3. Utilisez l'annotation `@Parameter` sur le paramètre `id` de la méthode `deleteAuthor` pour ajouter une description et
   un exemple.
4. Vérifiez dans Swagger UI :

- Que les paramètres de pagination apparaissent bien pour la liste des auteurs.
- Qu'une icône de cadenas apparaît à côté de l'endpoint `DELETE /api/authors/{id}`, indiquant qu'il est sécurisé.
- Que la description du paramètre `id` est visible.

### Correction exercice 8 {collapsible="true"}

**1 & 2 & 3. Modifications dans `AuthorController.java`**

```java
// package fr.formation.spring.bibliotech.api;

import io.swagger.v3.oas.annotations.Parameter;
import io.swagger.v3.oas.annotations.security.SecurityRequirement;
import org.springframework.data.domain.Page;
import org.springframework.data.domain.Pageable;
// ... autres imports

@Tag(name = "Auteurs", description = "API pour la gestion des auteurs")
@RestController
@RequestMapping("/api/authors")
public class AuthorController {
    // ...

    @Operation(summary = "Récupère une liste paginée d'auteurs")
    @GetMapping
    public Page<AuthorDto> getAllAuthors(Pageable pageable) {
        return this.authorRepository.findAll(pageable)
                .map(this.authorMapper::toDto);
    }

    // ...

    @Operation(summary = "Supprime un auteur",
            description = "Nécessite des droits d'administrateur.")
    @SecurityRequirement(name = "bearerAuth") // Endpoint sécurisé
    @ApiResponses(value = {
            @ApiResponse(responseCode = "204", description = "Auteur supprimé"),
            @ApiResponse(responseCode = "404", description = "Auteur non trouvé"),
            @ApiResponse(responseCode = "401", description = "Non autorisé"),
            @ApiResponse(responseCode = "403", description = "Accès refusé")
    })
    @DeleteMapping("/{id}")
    public ResponseEntity<Void> deleteAuthor(
            @Parameter(description = "ID de l'auteur à supprimer.", required = true, example = "1")
            @PathVariable Long id) {
        if (authorRepository.existsById(id)) {
            authorRepository.deleteById(id);
            return ResponseEntity.noContent().build();
        }
        return ResponseEntity.notFound().build();
    }
}
```

**4. Vérification dans Swagger UI**

- Pour `GET /api/authors`, les champs `page`, `size` et `sort` sont maintenant disponibles.
- L'endpoint `DELETE /api/authors/{id}` a une icône de cadenas. Si vous essayez de l'exécuter sans avoir configuré un
  token via le bouton "Authorize", vous obtiendrez probablement une erreur `401 Unauthorized` ou `403 Forbidden` (si
  Spring Security était configuré).
- En dépliant cet endpoint, le paramètre `id` a bien la description que nous avons fournie.

### Auto-évaluation

1. **(QCM)** Comment peut-on définir un titre et une description globale pour toute l'API dans la documentation
   Swagger ?
   a) En annotant la classe principale avec `@OpenAPIDefinition`.
   b) En créant un `Bean` de type `OpenAPI` dans une classe de configuration.
   c) En modifiant le fichier `application.properties`.
   d) En annotant chaque contrôleur avec `@ApiInfo`.

2. **_ (Question ouverte)_** Quel objet de Spring Data JPA faut-il ajouter comme paramètre à une méthode de contrôleur
   pour activer facilement la pagination ?

3. **(QCM)** Pour documenter un endpoint comme étant protégé par une authentification de type Bearer Token, quelle est
   l'approche la plus courante avec `springdoc` ?
   a) Ajouter `@Secured` sur la méthode.
   b) Définir un `SecurityScheme` dans la configuration `OpenAPI` et utiliser `@SecurityRequirement` sur la méthode.
   c) Créer un `Filter` de sécurité personnalisé.
   d) L'ajouter manuellement dans la description de l'annotation `@Operation`.

4. **_ (Question ouverte)_** À quoi sert l'annotation `@Parameter` ? Donnez un exemple d'utilisation.

5. **_ (Question ouverte)_** Si vous configurez un `SecurityScheme` globalement, quel bouton apparaît dans l'interface
   Swagger UI pour permettre aux utilisateurs de s'authentifier ?

### Conclusion

Vous avez appris à maîtriser les aspects avancés de la documentation d'API. Vous ne vous contentez plus de générer une
documentation basique, vous êtes capable de la sculpter pour qu'elle soit une ressource complète, professionnelle et
parfaitement adaptée aux besoins de votre projet. Vous savez comment configurer les métadonnées, documenter des cas
d'usage réels comme la pagination et la sécurité, et guider l'utilisateur avec des descriptions détaillées.

Votre API est maintenant non seulement bien conçue, mais aussi parfaitement expliquée. C'est le signe d'une application
de haute qualité. Dans le dernier chapitre de contenu, nous aborderons quelques patterns REST avancés pour gérer des
scénarios plus complexes, comme les recherches et les actions qui ne correspondent pas au modèle CRUD standard.