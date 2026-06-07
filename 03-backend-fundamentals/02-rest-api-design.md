# REST API Design

`[Entry]`

## What is REST

REST (Representational State Transfer) is a set of conventions for designing HTTP APIs. It is not a protocol. It is a style. The conventions are simple:

1. Resources are identified by URLs (`/users`, `/orders`)
2. HTTP methods describe the action (GET, POST, PUT, DELETE)
3. The server returns standard HTTP status codes
4. JSON is the default format

## URL Structure

Name resources with nouns, not verbs. The HTTP method is the verb.

```
GET    /users           List users
GET    /users/123       Get user 123
POST   /users           Create a user
PUT    /users/123       Replace user 123
PATCH  /users/123       Update user 123 partially
DELETE /users/123       Delete user 123
```

Nested resources:

```
GET    /users/123/orders          List orders for user 123
GET    /users/123/orders/456      Get order 456 for user 123
POST   /users/123/orders          Create an order for user 123
```

Keep nesting shallow. One level deep is fine. Two levels is the maximum. Beyond that, consider promoting the nested resource to a top-level resource with a filter:

```
GET /orders?userId=123
```

## HTTP Methods

| Method | Action | Idempotent | Safe |
|--------|--------|------------|------|
| GET | Read | Yes | Yes |
| POST | Create | No | No |
| PUT | Replace entirely | Yes | No |
| PATCH | Partial update | No | No |
| DELETE | Remove | Yes | No |

**Idempotent:** Calling it multiple times produces the same result. PUT and DELETE are idempotent. POST is not (each call may create a new resource).

**Safe:** Does not modify data. Only GET is safe.

## Status Codes

Use the correct status code. It is how clients understand what happened.

**Success:**

| Code | When |
|------|------|
| 200 OK | Successful GET, PUT, PATCH |
| 201 Created | Successful POST (new resource created) |
| 204 No Content | Successful DELETE (no body needed) |

**Client errors:**

| Code | When |
|------|------|
| 400 Bad Request | Invalid input (malformed JSON, validation failure) |
| 401 Unauthorized | Missing or invalid authentication |
| 403 Forbidden | Authenticated but not allowed |
| 404 Not Found | Resource does not exist |
| 409 Conflict | Resource already exists or state conflict |
| 422 Unprocessable Entity | Valid JSON but semantically invalid |

**Server errors:**

| Code | When |
|------|------|
| 500 Internal Server Error | Unexpected server failure |

Do not use 500 for validation errors. Do not use 200 for errors. Let the status code carry the meaning.

## Request and Response Format

**POST /users -- Create:**

Request:
```json
{
    "name": "Alice",
    "email": "alice@example.com"
}
```

Response (201 Created):
```json
{
    "id": "123",
    "name": "Alice",
    "email": "alice@example.com",
    "createdAt": "2026-01-15T10:30:00Z"
}
```

**GET /users/123 -- Read:**

Response (200 OK):
```json
{
    "id": "123",
    "name": "Alice",
    "email": "alice@example.com",
    "createdAt": "2026-01-15T10:30:00Z"
}
```

**GET /users -- List:**

Response (200 OK):
```json
[
    { "id": "123", "name": "Alice", "email": "alice@example.com" },
    { "id": "456", "name": "Bob", "email": "bob@example.com" }
]
```

## Pagination

For list endpoints, return paginated results:

```
GET /users?page=2&size=20
```

Response:
```json
{
    "data": [...],
    "page": 2,
    "size": 20,
    "total": 150
}
```

Always paginate list endpoints. Even if the list is small today, it will grow. Adding pagination later is a breaking change.

## Error Response Format

Use a consistent error structure:

```json
{
    "error": "validation_error",
    "message": "Email is invalid",
    "details": [
        { "field": "email", "issue": "must contain @" }
    ]
}
```

Consistency matters. Clients parse errors programmatically. A predictable structure makes error handling reliable.

## Versioning

Prefix URLs with a version:

```
GET /v1/users
POST /v1/users
```

When you make breaking changes, create a new version (`/v2/users`). Maintain the old version until clients migrate.

## Key Takeaways

- URLs are nouns. HTTP methods are verbs. `GET /users` not `GET /getUsers`.
- Use correct status codes. 201 for creation, 400 for bad input, 404 for missing resources.
- Paginate all list endpoints from the start.
- Use a consistent error response format.
- Version your API from day one.
