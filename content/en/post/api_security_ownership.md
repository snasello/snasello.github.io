---
date: '2024-12-28T16:46:47+01:00'
draft: false
title: 'Securing Our APIs: Verifying Ownership'
featured_image: "/images/api_security_ownership.png"
tags: ["dev","api","security"]
---

Security is a vast topic which, unfortunately, is often someone else's problem.
As we often hear, security is everyone's business!
So, let's set aside firewalls and other infrastructure components, and talk about code and APIs.

Let's set the context: an API with authentication, where the user accesses data that belongs to them.
For example, we are on an e-commerce site, and we display their orders.

On the API, we will have two endpoints:

```
GET /orders
GET /orders/{order-id}
```

For the first one, to retrieve data from the database, we will have a SQL query like:

```sql
SELECT * FROM ORDERS WHERE customer_id = ?
```

Nothing special to say here, it will correctly return the client's orders!

Now let's look at the second endpoint, we will generally have a SQL query like:

```sql
SELECT * FROM ORDERS WHERE order_id = ?
```

Except there is no verification on the client's identifier.
A connected client could therefore retrieve the orders of another client, along with the associated data (address, emails, etc.).
This is even simpler if the order identifier is an increment.
An attacker would just need to run a simple loop using their authentication token.

To remedy this, we can add a condition in the WHERE clause:

```sql
SELECT * FROM ORDERS WHERE order_id = ? AND customer_id = ?
```

From an API perspective, this means the order doesn't exist if one tries to access another client's order.
The API will return an HTTP 404 (Not Found) code.

Another possibility would be to keep the first SQL query and then verify in the code.

The SQL query:

```sql
SELECT * FROM ORDERS WHERE order_id = ?
```

In the code (pseudo code):

```java
Order order = ordersRepository.findById(orderId);
if (order == null) {
    return 404;
}
if (order.customerId != customerId) {
    return 403;
}
return order;
```

We can now ask whether it is better to return a 404 or a 403.
From a security standpoint, it is preferable to leave as few clues as possible for an attacker.
The 404 code has the advantage of not indicating whether the identifier exists or not.

In our example, I think it is the most relevant.
But in other contexts, returning a 403 code could be advisable.
For instance, in a DMS (Document Management System): a user shares a link with another user, the second one doesn't have rights.
Here, I think the 403 code is relevant as it allows displaying a page to request rights from the owner.

## Conclusion
When we code our APIs, we cannot rely solely on authentication.
It is important to verify data ownership to avoid exposing ourselves to data leaks.