---
date: '2024-12-28T16:46:47+01:00'
draft: false
title: 'Sécurisons nos API : vérifier l’appartenance'
featured_image: "/images/api_security_ownership.png"
tags: ["dev","api","security"]
---

La sécurité est un vaste sujet qui, malheureusement, est souvent le problème d’un “autre”. 
Comme on entend fréquemment, la sécurité est l’affaire de tous !
Alors, mettons de côté les firewalls et autres composants d’infrastructure, et parlons de code et d’API.

Posons le contexte : une API avec une authentification, où l’utilisateur accède à une donnée qui lui appartient.
Par exemple, nous sommes sur un site e-commerce et nous affichons ses commandes.

Sur l’api, nous aurons deux endpoints :

```
GET /orders
GET /orders/{order-id}
```

Sur le premier, pour récupérer les données dans la base, nous aurons une requête SQL de type :

```sql
SELECT * FROM ORDERS WHERE customer_id = ?
```

Rien de spécial à dire ici, cela renverra bien les commandes du client !

Prenons le second endpoint, nous aurons généralement une requête SQL de type :
```sql
SELECT * FROM ORDERS WHERE order_id = ?
```

Sauf qu'il n’y a pas de vérification sur l’identifiant du client.
Un client connecté pourrait donc récupérer les commandes d’un autre client, et les données qui vont avec (adresse, mails, etc.).
C’est d’autant plus simple si l’identifiant de la commande est un incrément. Il suffira pour un attaquant de faire une simple boucle utilisant son jeton d’authentification.

Pour y remédier, nous pouvons ajouter une condition dans la clause where :

```sql
SELECT * FROM ORDERS WHERE order_id = ? AND customer_id = ?
```

D’un point de vue API, cela revient à dire que la commande n’existe pas si l’on tente d’accéder à la commande d’un autre client.
L’API renverra un code HTTP 404 (Not Found).

Une autre possibilité serait de garder la première requête SQL et vérifier ensuite dans le code.

La requête SQL :

```sql
SELECT * FROM ORDERS WHERE order_id = ?
```
Au niveau code (pseudo code) :

```
Order order = ordersRepository.findById(orderId);
if (order == null) {
   return 404;
}
if (order.customerId != customerId) {
   return 503;
}
return order;
```

On peut maintenant se demander s'il est préférable de renvoyer une 404 ou une 403.
D’un point de vue sécurité, il est préférable de laisser le moins d’indice possible à un attaquant.
Le code 404 a l’avantage de ne pas indiquer si l’identifiant existe ou pas.

Dans notre exemple, je pense que c’est le plus pertinent.
Mais dans d’autres contextes, renvoyer un code 403 pourrait être judicieux.
Prenons l’exemple d’une GED : un utilisateur partage un lien avec un autre, le second n’a pas les droits.
Ici, je pense que le code 403 est pertinent, cela permet d’afficher une page pour demander les droits au propriétaire.

## Conclusion

Quand nous codons nos API, nous ne pouvons pas nous reposer uniquement sur l’authentification.
Il est important de vérifier l’appartenance de la donnée afin d’éviter de nous exposer à une fuite de donnée.