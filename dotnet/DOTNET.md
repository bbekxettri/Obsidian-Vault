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

---
## HTTP Methods Deep Dive

Getting HTTP methods right is fundamental. Here’s the complete picture.

### GET – Retrieve Resources

Retrieve a resource without modifying it. **Must be safe and idempotent**.

```
app.MapGet("/api/users", async (UserService service) =>{    var users = await service.GetAllAsync();    return Results.Ok(users);});
app.MapGet("/api/users/{id:int}", async (int id, UserService service) =>{    var user = await service.GetByIdAsync(id);    return user is null ? Results.NotFound() : Results.Ok(user);});
```

**Response**: `200 OK` with resource body, or `404 Not Found` if it doesn’t exist.

### POST – Create Resources

Create a new resource. **Not idempotent**—calling it twice creates two resources.

```
app.MapPost("/api/users", async (CreateUserRequest request, UserService service) =>{    var user = await service.CreateAsync(request);    return Results.Created($"/api/users/{user.Id}", user);});
```

**Key points**:

- Return `201 Created`, not `200 OK`
- Include `Location` header pointing to the new resource
- Return the created resource in the body

### PUT – Replace Resources

Replace an entire resource. **Must be idempotent**—calling it multiple times produces the same result.

```
app.MapPut("/api/users/{id:int}", async (int id, UpdateUserRequest request, UserService service) =>{    var user = await service.GetByIdAsync(id);    if (user is null)        return Results.NotFound();
    await service.UpdateAsync(id, request);    return Results.NoContent();});
```

**Key points**:

- Client sends the **complete** resource representation
- Missing fields are set to null/default (not ignored)
- Return `204 No Content` on success (or `200 OK` with updated resource)

### PATCH – Partial Updates

Update specific fields of a resource. Use when you only want to modify some properties.

```
app.MapPatch("/api/users/{id:int}", async (int id, JsonPatchDocument<User> patchDoc, UserService service) =>{    var user = await service.GetByIdAsync(id);    if (user is null)        return Results.NotFound();
    patchDoc.ApplyTo(user);    await service.UpdateAsync(id, user);    return Results.NoContent();});
```

Or with a simpler DTO approach:

```
public record PatchUserRequest(string? Name, string? Email);
app.MapPatch("/api/users/{id:int}", async (int id, PatchUserRequest request, UserService service) =>{    var user = await service.GetByIdAsync(id);    if (user is null)        return Results.NotFound();
    // Only update provided fields    if (request.Name is not null) user.Name = request.Name;    if (request.Email is not null) user.Email = request.Email;
    await service.UpdateAsync(id, user);    return Results.NoContent();});
```

### DELETE – Remove Resources

Delete a resource. **Should be idempotent**—deleting something twice shouldn’t error (the second request can return 404).

```
app.MapDelete("/api/users/{id:int}", async (int id, UserService service) =>{    var user = await service.GetByIdAsync(id);    if (user is null)        return Results.NotFound();
    await service.DeleteAsync(id);    return Results.NoContent();});
```

**Response**: `204 No Content` on success.
___
## HTTP Status Codes Done Right

| Code    | Name                  | When to Use                                    |
| ------- | --------------------- | ---------------------------------------------- |
| **200** | OK                    | Returning data successfully                    |
| **201** | Created               | New resource created (include Location header) |
| **204** | No Content            | Success with no body (DELETE, some PUT/PATCH)  |
| **202** | Accepted              | Async operation accepted for processing        |
| **400** | Bad Request           | Malformed request (bad JSON, wrong types)      |
| **401** | Unauthorized          | Not authenticated                              |
| **403** | Forbidden             | Authenticated but not authorized               |
| **404** | Not Found             | Resource doesn’t exist                         |
| **409** | Conflict              | Request conflicts with current state           |
| **422** | Unprocessable Entity  | Validation errors                              |
| **429** | Too Many Requests     | Rate limit exceeded                            |
| **500** | Internal Server Error | Unexpected server error                        |
| **502** | Bad Gateway           | Invalid upstream response                      |
| **503** | Service Unavailable   | Server temporarily unavailable                 |
| **504** | Gateway Timeout       | Upstream service timeout                       |

---

