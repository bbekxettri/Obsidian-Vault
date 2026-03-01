## The Six REST Constraints
### 1. Client-Server Separation
### 2. Statelessness
### 3. Cacheability
### 4. Uniform Interface
### 5. Layered System
### 6. Code on Demand (Optional)

## RESTful API Design Best Practices

### Use Nouns, Not Verbs

URIs identify resources. Use nouns. The HTTP method provides the verb.

|❌ Bad (Verbs in URI)|✅ Good (Resource Nouns)|
|---|---|
|`GET /api/getUsers`|`GET /api/users`|
|`POST /api/createUser`|`POST /api/users`|
|`PUT /api/updateUser/42`|`PUT /api/users/42`|
|`DELETE /api/deleteUser/42`|`DELETE /api/users/42`|
|`GET /api/getUserOrders/42`|`GET /api/users/42/orders`|

### Use Plural Nouns for Collections

Always use plural nouns for resource collections. It’s consistent and reads naturally.

| ❌ Inconsistent | ✅ Consistent (Plural) |
| -------------- | --------------------- |
| `/api/user`    | `/api/users`          |
| `/api/user/42` | `/api/users/42`       |
| `/api/product` | `/api/products`       |
| `/api/order`   | `/api/orders`         |
