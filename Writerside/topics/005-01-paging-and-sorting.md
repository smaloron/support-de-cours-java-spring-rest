# Chapitre 6 : Patterns Avancés - Gérer de Grands Ensembles de Données (L'Essentiel)

Jusqu'à présent, notre API "BiblioTech" fonctionne à merveille avec notre petit jeu de données de test. Mais imaginez
une vraie bibliothèque avec des dizaines de milliers de livres. Si un client appelle `GET /api/books`, doit-on vraiment
lui envoyer une réponse JSON de plusieurs mégaoctets contenant 50 000 objets ? La réponse est un non retentissant. Cela
saturerait le réseau, ferait planter le navigateur du client et mettrait notre serveur à genoux. Il nous faut une
stratégie pour servir de grands ensembles de données de manière intelligente.

### Objectifs Pédagogiques

À la fin de cette partie, vous serez capable de :

- Comprendre les problèmes liés au retour de collections de données trop volumineuses.
- Mettre en œuvre la pagination pour retourner les données par "pages" ou "morceaux".
- Utiliser l'objet `Pageable` de Spring Data JPA pour simplifier la pagination.
- Implémenter le tri pour permettre aux clients de classer les résultats selon leurs besoins.
- Interpréter la structure de réponse paginée fournie par Spring.

### Introduction : Le Bibliothécaire Efficace

Imaginez que vous demandez à un bibliothécaire "tous les livres sur l'histoire". Un bibliothécaire inefficace
commencerait à empiler des milliers de livres sur le comptoir jusqu'à ce qu'il s'effondre. Un bibliothécaire efficace,
lui, vous dirait : "Bien sûr. Voici les 10 premiers, rangés par ordre alphabétique. Dites-moi si vous voulez voir les 10
suivants."

Dans le monde des API, nous devons être ce bibliothécaire efficace. La **pagination** est la technique qui consiste à
diviser une grande collection de résultats en plus petits morceaux (des pages). Le **tri** est la capacité à organiser
les résultats de chaque page selon un critère précis (par titre, par date, etc.). Spring Boot rend ces deux opérations
remarquablement simples.

### 1. La Pagination avec Spring Data JPA

L'écosystème Spring Data nous offre un outil magique pour gérer la pagination et le tri : l'interface `Pageable`. En
l'ajoutant simplement comme paramètre à une méthode de contrôleur, Spring va automatiquement chercher les paramètres de
pagination dans l'URL de la requête.

Les paramètres standards sont :

- `page` : Le numéro de la page que l'on souhaite récupérer (commence à 0).
- `size` : Le nombre d'éléments à inclure dans une page.

<procedure title="Ajout de la pagination au `BookController`">

Modifions notre méthode `getAllBooks` (que nous avions transformée en `searchBooks`). Pour la clarté, revenons à un
endpoint de base listant tous les livres, mais de manière paginée.

1. **Le contrôleur s'adapte**

```java
// Dans BookController.java

import org.springframework.data.domain.Page;
import org.springframework.data.domain.Pageable;

// ...

@Operation(summary = "Récupère une liste paginée de livres")
@GetMapping
public Page<BookDto> getAllBooks(Pageable pageable) {
    // La méthode findAll(pageable) de Spring Data retourne directement
    // un objet Page. C'est tout ce qu'il y a à faire !
    Page<Book> bookPage = this.bookRepository.findAll(pageable);

    // On mappe la page d'entités en une page de DTOs.
    return bookPage.map(this.bookMapper::toDto);
}
```

2. **Tester avec une requête HTTP**

Essayons de récupérer la première page, avec 2 livres par page.

```http
### Récupérer la première page de livres (2 par page)
GET http://localhost:8080/api/books?page=0&size=2
Accept: application/json
```

3. **Analyser la réponse**

La réponse n'est plus un simple tableau JSON. C'est un objet JSON qui contient les données de la page (`content`) ainsi
que les métadonnées de pagination.

```json
{
  "content": [
    {
      "id": 1,
      "title": "Harry Potter and the Sorcerer's Stone",
      "isbn": "978-0439708180"
      // ...
    },
    {
      "id": 2,
      "title": "The Hobbit",
      "isbn": "978-0345339683"
      // ...
    }
  ],
  "pageable": {
    "pageNumber": 0,
    "pageSize": 2,
    "sort": {
      "unsorted": true,
      "sorted": false,
      "empty": true
    }
    // ...
  },
  "totalPages": 2,
  // Il y a 2 pages au total
  "totalElements": 3,
  // Il y a 3 livres au total dans la BDD
  "last": false,
  // Ce n'est pas la dernière page
  "size": 2,
  "number": 0,
  // Numéro de la page actuelle
  "numberOfElements": 2,
  // Nombre d'éléments sur cette page
  "first": true,
  // C'est la première page
  "empty": false
}
```

Ces métadonnées sont extrêmement utiles pour le client (par exemple, une application web) afin de construire une
interface de pagination (ex: "Page 1 sur 2", boutons "Précédent"/"Suivant").

</procedure>

### 2. Le Tri

Le tri est un compagnon naturel de la pagination. Le client peut le spécifier via le paramètre de requête `sort`.

- Le format est `sort=propriete,direction`.
- Exemple : `sort=title,asc` pour trier par titre en ordre ascendant.
- Exemple : `sort=publicationDate,desc` pour trier par date de publication en ordre descendant.
- On peut même combiner plusieurs critères : `sort=lastName,asc&sort=firstName,asc`.

Bonne nouvelle : nous n'avons **rien à changer dans notre code** ! L'objet `Pageable` gère le tri automatiquement.

<procedure title="Tester le tri">

```http
### Récupérer les livres triés par titre (ascendant)
GET http://localhost:8080/api/books?size=5&sort=title,asc

### Récupérer les livres triés par date de publication (plus récent d'abord)
GET http://localhost:8080/api/books?size=5&sort=publicationDate,desc
```

Le serveur retournera les livres dans l'ordre demandé.

</procedure>

<div class="alert alert-warning" title="Attention aux propriétés de tri">
<p>Le client doit utiliser les noms des propriétés de l'<b>entité JPA</b> (<code>publicationDate</code>), pas forcément ceux du DTO. Si les noms diffèrent, vous pourriez avoir besoin d'une configuration plus avancée. Pour des raisons de sécurité, il est aussi recommandé de valider les champs sur lesquels le tri est autorisé pour éviter que des utilisateurs malicieux tentent de trier sur des champs sensibles.</p>
</div>

### Exercice 11 : Paginer et Trier les Auteurs

Appliquez ces concepts à notre `AuthorController`.

**Énoncé :**

1. Modifiez la méthode `getAllAuthors` dans `AuthorController` pour qu'elle accepte un `Pageable` en paramètre.
2. La méthode doit maintenant retourner un `Page<AuthorDto>`.
3. Utilisez `authorRepository.findAll(pageable)` et mappez le résultat en `Page<AuthorDto>`.
4. Testez votre nouvel endpoint avec les requêtes suivantes :
    - Récupérer la première page de 1 auteur.
    - Récupérer tous les auteurs (en utilisant une taille de page assez grande) triés par nom de famille en ordre
      descendant (`lastName,desc`).

### Correction exercice 11 {collapsible="true"}

**1, 2, 3. `AuthorController.java` modifié**

```java
// package fr.formation.spring.bibliotech.api;

import org.springframework.data.domain.Page;
import org.springframework.data.domain.Pageable;
// ...

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
}
```

**4. Requêtes de test**

```http
### Récupérer la première page de 1 auteur
GET http://localhost:8080/api/authors?page=0&size=1

### Récupérer les auteurs triés par nom de famille (Z->A)
GET http://localhost:8080/api/authors?size=5&sort=lastName,desc
```

Vous devriez obtenir des réponses JSON structurées avec l'objet `Page`, contenant les auteurs demandés dans le bon
ordre.

### Auto-évaluation

1. **(QCM)** Quel objet Spring faut-il ajouter comme paramètre de méthode dans un contrôleur pour activer la pagination
   et le tri ?
   a) `Pagination`
   b) `PageRequest`
   c) `Pageable`
   d) `Sortable`

2. **_ (Question ouverte)_** Citez deux raisons pour lesquelles il est indispensable de paginer les réponses d'API qui
   retournent de grandes collections.

3. **(QCM)** Dans une réponse paginée de Spring Data, où se trouve le tableau des données de la page actuelle ?
   a) Dans le champ `data`.
   b) Dans le champ `content`.
   c) Dans le champ `page`.
   d) À la racine de l'objet JSON.

4. **_ (Question ouverte)_** Quelle est la syntaxe du paramètre de requête pour demander la troisième page de résultats,
   avec 10 éléments par page ?

5. **_ (Question ouverte)_** Comment demanderiez-vous à l'API de trier les résultats par prénom (`firstName`) en ordre
   ascendant, puis par nom de famille (`lastName`) en ordre descendant ?

### Conclusion

Vous venez de doter votre API d'une capacité essentielle pour gérer des applications du monde réel. En maîtrisant la
pagination et le tri, vous garantissez que votre API reste performante et utilisable, même avec des millions
d'enregistrements. Vous savez maintenant comment servir des données de manière efficace et comment donner au client le
contrôle sur la façon dont il les reçoit.

Cependant, la vie d'une API ne s'arrête pas à sa première version. Elle va évoluer. Comment gérer ces évolutions sans
casser les applications qui l'utilisent ? Et comment rendre l'API encore plus "intelligente" pour guider le client ?
C'est ce que nous verrons dans la partie suivante avec le versioning et HATEOAS.