# HTTP and Web Servers



## What is HTTP

HTTP (HyperText Transfer Protocol) is the protocol that powers the web. It is a request-response protocol: a client sends a request, a server sends a response.

Every web interaction follows this pattern:

```
Client -> Request -> Server -> Response -> Client
```

## HTTP Request

An HTTP request has three parts:

**1. Method and URL:**

```
GET /users/123 HTTP/1.1
```

The method tells the server what to do. The URL identifies the resource.

Common methods:

| Method | Purpose | Has Body |
|--------|---------|----------|
| GET | Retrieve a resource | No |
| POST | Create a resource | Yes |
| PUT | Replace a resource entirely | Yes |
| PATCH | Partially update a resource | Yes |
| DELETE | Remove a resource | No |

**2. Headers:**

```
Content-Type: application/json
Authorization: Bearer eyJhbGciOi...
Accept: application/json
```

Headers are metadata. They describe the request: what format the body is in, who is making the request, what format the client accepts in the response.

**3. Body (optional):**

```json
{
    "name": "Alice",
    "email": "alice@example.com"
}
```

The body contains data. Used with POST, PUT, and PATCH. GET and DELETE typically have no body.

## HTTP Response

An HTTP response also has three parts:

**1. Status code:**

```
HTTP/1.1 201 Created
```

The status code tells the client what happened. Key categories:

| Range | Meaning | Examples |
|-------|---------|----------|
| 2xx | Success | 200 OK, 201 Created, 204 No Content |
| 3xx | Redirection | 301 Moved Permanently, 304 Not Modified |
| 4xx | Client error | 400 Bad Request, 401 Unauthorized, 404 Not Found |
| 5xx | Server error | 500 Internal Server Error, 502 Bad Gateway |

**2. Headers:**

```
Content-Type: application/json
Cache-Control: no-cache
```

**3. Body:**

```json
{
    "id": "123",
    "name": "Alice",
    "email": "alice@example.com"
}
```

## What is a Web Server

A web server is a program that:

1. Listens for incoming HTTP requests on a port (e.g., port 8080)
2. Routes each request to the correct handler based on the method and URL
3. Executes the handler (your code)
4. Sends back an HTTP response

```
[Client] --HTTP Request--> [Web Server:8080] --route match--> [Handler] --HTTP Response--> [Client]
```

In Kotlin with Ktor, your application IS the web server. There is no external container like Tomcat. Ktor embeds the server engine (Netty by default) directly in your application.

## Statelessness

HTTP is stateless. Each request is independent. The server does not remember previous requests. State (like "this user is logged in") is managed through:

- **Tokens:** Client sends a token (JWT) in the Authorization header with each request.
- **Cookies:** Client sends cookies with each request. Server reads them.
- **Session IDs:** Server stores session data, client sends a session ID.

Statelessness is important for scaling. If requests are independent, you can add more servers behind a load balancer and any server can handle any request.

## Content Types

The `Content-Type` header tells the server and client what format the data is in:

| Content-Type | Format |
|--------------|--------|
| `application/json` | JSON (most common for APIs) |
| `application/xml` | XML |
| `text/html` | HTML |
| `text/plain` | Plain text |
| `application/octet-stream` | Binary data |

In a Kotlin backend, you almost always use `application/json`. Ktor's ContentNegotiation plugin handles this automatically.

## Key Takeaways

- HTTP is request-response. Client sends a request, server sends a response.
- Methods (GET, POST, PUT, DELETE) describe the action. URLs identify the resource.
- Status codes (2xx success, 4xx client error, 5xx server error) describe the result.
- A web server listens for requests, routes them, and sends responses.
- HTTP is stateless. Tokens or cookies manage state across requests.
