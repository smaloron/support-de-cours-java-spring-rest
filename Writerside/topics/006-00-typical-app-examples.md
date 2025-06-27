# Exemples d'Applications Types

Jusqu'à présent, nous avons travaillé sur un projet fil rouge, "BiblioTech". C'est un excellent cas d'étude, mais
comment ces concepts s'appliquent-ils à d'autres domaines ? Un blog, un site e-commerce ou un réseau social ont des
besoins et des logiques métier différents. Cette section a pour but de vous montrer comment adapter les patterns que
vous avez appris à divers types d'applications.

### Objectifs Pédagogiques

À la fin de cette partie, vous serez capable de :

- Identifier les ressources principales pour différents domaines d'application (e-commerce, blog, etc.).
- Imaginer des endpoints RESTful pour des logiques métier spécifiques.
- Transposer les concepts de sous-ressources et d'actions à de nouveaux contextes.
- Comprendre comment l'architecture REST s'adapte à une variété de problèmes.

### Introduction : Le Couteau Suisse de l'API

L'architecture REST n'est pas spécifique à un domaine. C'est un ensemble de principes universels de communication.
Pensez-y comme à un couteau suisse : les outils de base (lames, tournevis) sont toujours les mêmes, mais vous les
utilisez différemment selon que vous ayez besoin d'ouvrir une boîte de conserve ou de réparer des lunettes.

De la même manière, les verbes HTTP, les URIs basées sur les ressources, les DTOs et la gestion des erreurs sont les "
lames" de notre couteau suisse REST. Nous allons maintenant voir comment les appliquer à différents scénarios pour "
tailler" des solutions sur mesure.

---

### Cas d'étude 1 : Un Site E-commerce

Un site e-commerce est un cas d'usage très riche pour une API REST.

#### Ressources Principales {id="ressources-principales_1"}

- `/products` : Les produits en vente.
- `/customers` : Les clients.
- `/orders` : Les commandes passées.
- `/carts` : Les paniers d'achat.

#### Endpoints CRUD Classiques {id="endpoints-crud-classiques_1"}

- `GET /api/v1/products` : Lister tous les produits (avec pagination et recherche !).
- `GET /api/v1/products/{productId}` : Voir les détails d'un produit.
- `POST /api/v1/customers` : Créer un nouveau compte client (inscription).
- `GET /api/v1/customers/{customerId}` : Récupérer les informations d'un client (authentifié).

#### Sous-Ressources (relations)

- `GET /api/v1/customers/{customerId}/orders` : Lister toutes les commandes d'un client spécifique. C'est bien plus
  RESTful qu'un `GET /api/v1/orders?customerId=123`.
- `GET /api/v1/products/{productId}/reviews` : Obtenir tous les avis pour un produit.
- `POST /api/v1/products/{productId}/reviews` : Poster un nouvel avis pour un produit.

#### Actions (logique métier)

- `POST /api/v1/carts/{cartId}/items` : Ajouter un article au panier. Le corps de la requête contiendrait
  `{ "productId": 456, "quantity": 2 }`.
- `DELETE /api/v1/carts/{cartId}/items/{itemId}` : Supprimer un article du panier.
- `POST /api/v1/carts/{cartId}/checkout` : **C'est l'action clé !** Ce `POST` déclenche tout le processus de passage en
  caisse. Il ne "crée" rien au sens strict dans le panier, mais il transforme l'état du panier en une commande. La
  réponse pourrait être un `303 See Other` redirigeant vers la nouvelle ressource commande (
  `/api/v1/orders/{newOrderId}`) ou un `201 Created` avec la commande en corps de réponse.
- `POST /api/v1/orders/{orderId}/shipment` : Déclencher l'expédition d'une commande (côté back-office).

---

### Cas d'étude 2 : Un Blog

Un blog est un exemple plus simple mais qui illustre bien certains concepts.

#### Ressources Principales {id="ressources-principales_2"}

- `/posts` : Les articles du blog.
- `/authors` : Les auteurs (si multi-auteurs).
- `/tags` : Les étiquettes ou catégories.
- `/comments` : Les commentaires.

#### Endpoints CRUD Classiques {id="endpoints-crud-classiques_2"}

- `GET /api/v1/posts` : Lister tous les articles (les plus récents en premier).
- `GET /api/v1/posts/{postId}` : Lire un article.
- `POST /api/v1/posts` : Créer un nouvel article (pour l'admin).
- `PUT /api/v1/posts/{postId}` : Mettre à jour un article.

#### Sous-Ressources

- `GET /api/v1/posts/{postId}/comments` : Lister tous les commentaires d'un article.
- `POST /api/v1/posts/{postId}/comments` : Ajouter un nouveau commentaire à un article. Le corps de la requête
  contiendrait le texte et le nom du commentateur.
- `GET /api/v1/authors/{authorId}/posts` : Lister tous les articles d'un auteur spécifique.
- `GET /api/v1/tags/{tagName}/posts` : Lister tous les articles ayant une certaine étiquette.

#### Actions

- `POST /api/v1/posts/{postId}/publication` : Publier un article qui est actuellement en brouillon.
- `DELETE /api/v1/posts/{postId}/publication` : Dé-publier un article et le remettre en brouillon.
- `POST /api/v1/comments/{commentId}/approval` : Approuver un commentaire en attente de modération.
- `POST /api/v1/posts/{postId}/like` : "Aimer" un article. C'est une action idempotente si on la conçoit bien (un
  deuxième appel n'ajoute pas un deuxième "like"). Un `PUT` pourrait aussi être discuté ici.

---

### Cas d'étude 3 : Un Réseau Social (simplifié)

Un réseau social est basé sur les relations et les interactions, ce qui en fait un excellent candidat pour les patterns
avancés.

#### Ressources Principales

- `/users` : Les utilisateurs.
- `/posts` : Les publications (similaire à un blog).
- `/groups` : Les groupes de discussion.

#### Endpoints CRUD Classiques

- `GET /api/v1/users/{userId}` : Voir le profil d'un utilisateur.
- `GET /api/v1/posts` : Obtenir le fil d'actualité (logique de recherche/filtrage très complexe ici !).
- `POST /api/v1/posts` : Créer une nouvelle publication.

#### Sous-Ressources et Relations

C'est ici que ça devient intéressant.

- `GET /api/v1/users/{userId}/friends` : Lister les amis d'un utilisateur.
- `GET /api/v1/users/{userId}/posts` : Lister les publications d'un utilisateur spécifique.
- `GET /api/v1/groups/{groupId}/members` : Lister les membres d'un groupe.

#### Actions et Gestion des Relations

Comment gérer les demandes d'amitié ? Ce n'est pas un CRUD simple.

- `POST /api/v1/friend-requests` : Envoyer une demande d'amitié. Le corps contiendrait `{"toUserId": 789}`.
- `GET /api/v1/friend-requests?status=pending` : Voir ses demandes d'amitié en attente.
- `POST /api/v1/friend-requests/{requestId}/acceptance` : Accepter une demande d'amitié.
- `DELETE /api/v1/friend-requests/{requestId}` : Refuser ou annuler une demande.

Autre exemple : suivre un utilisateur.

- `PUT /api/v1/users/{userId}/followers/{followerId}` : Un utilisateur (`followerId`) décide de suivre un autre
  utilisateur (`userId`). `PUT` est parfait ici : la relation est créée si elle n'existe pas, et rien ne se passe si
  elle existe déjà (idempotence). Le corps de la requête peut être vide.
- `DELETE /api/v1/users/{userId}/followers/{followerId}` : Ne plus suivre l'utilisateur.

### Conclusion

Comme vous pouvez le voir, les principes REST sont incroyablement flexibles. Le défi n'est pas technique (Spring Boot
facilite l'implémentation), mais conceptuel :

1. **Identifier les bonnes ressources** : Quels sont les "noms" importants de votre domaine ?
2. **Penser en termes de relations et de hiérarchies** : Quelles ressources appartiennent à d'autres ? C'est la porte d'
   entrée aux sous-ressources.
3. **Identifier les "verbes" métier** : Quelles sont les actions qui changent l'état de votre système mais qui ne sont
   pas un simple CRUD ? C'est la porte d'entrée aux endpoints d'action.

En vous exerçant à analyser des problèmes sous cet angle, vous développerez une intuition pour concevoir des API
claires, expressives et logiques, quel que soit le domaine d'application. Vous avez maintenant les clés pour appliquer
ce que vous avez appris bien au-delà de notre projet "BiblioTech".

La prochaine et dernière section sera une conclusion générale, qui récapitulera le parcours et vous donnera des pistes
pour la suite de votre apprentissage.