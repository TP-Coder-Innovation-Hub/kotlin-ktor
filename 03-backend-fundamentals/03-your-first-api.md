# Your First API with Ktor

`[Mid]`

## Setup

Create a Ktor project. Add these dependencies to `build.gradle.kts`:

```kotlin
plugins {
    kotlin("jvm") version "2.1.0"
    kotlin("plugin.serialization") version "2.1.0"
    id("io.ktor.plugin") version "3.1.0"
    application
}

dependencies {
    implementation("io.ktor:ktor-server-core:3.1.0")
    implementation("io.ktor:ktor-server-netty:3.1.0")
    implementation("io.ktor:ktor-server-content-negotiation:3.1.0")
    implementation("io.ktor:ktor-serialization-kotlinx-json:3.1.0")
    implementation("io.ktor:ktor-server-status-pages:3.1.0")
    implementation("org.jetbrains.kotlinx:kotlinx-serialization-json:1.8.0")
}

application {
    mainClass.set("com.example.ApplicationKt")
}
```

## Data Models

Define your request and response types:

```kotlin
package com.example.models

import kotlinx.serialization.Serializable

@Serializable
data class User(
    val id: String,
    val name: String,
    val email: String
)

@Serializable
data class CreateUserRequest(
    val name: String,
    val email: String
)
```

`@Serializable` enables automatic JSON conversion via kotlinx.serialization.

## In-Memory Repository

For now, store data in memory. Database integration comes later.

```kotlin
package com.example.repository

import com.example.models.User
import com.example.models.CreateUserRequest
import java.util.concurrent.ConcurrentHashMap

class UserRepository {
    private val users = ConcurrentHashMap<String, User>()

    fun findAll(): List<User> = users.values.toList()

    fun findById(id: String): User? = users[id]

    fun create(request: CreateUserRequest): User {
        val user = User(
            id = java.util.UUID.randomUUID().toString(),
            name = request.name,
            email = request.email
        )
        users[user.id] = user
        return user
    }

    fun delete(id: String): Boolean = users.remove(id) != null
}
```

`ConcurrentHashMap` is thread-safe. Multiple requests can read and write concurrently without corruption.

## Application Entry Point

```kotlin
package com.example

import com.example.repository.UserRepository
import io.ktor.http.*
import io.ktor.serialization.kotlinx.json.*
import io.ktor.server.application.*
import io.ktor.server.engine.*
import io.ktor.server.netty.*
import io.ktor.server.plugins.contentnegotiation.*
import io.ktor.server.plugins.statuspages.*
import io.ktor.server.request.*
import io.ktor.server.response.*
import io.ktor.server.routing.*
import kotlinx.serialization.json.Json

fun main() {
    embeddedServer(Netty, port = 8080, host = "0.0.0.0") {
        configureSerialization()
        configureStatusPages()
        configureRouting()
    }.start(wait = true)
}
```

Step by step:

1. `embeddedServer(Netty, ...)` starts a web server on port 8080 using the Netty engine.
2. `configureSerialization()` installs JSON serialization.
3. `configureStatusPages()` installs error handling.
4. `configureRouting()` defines the routes.
5. `.start(wait = true)` starts the server and blocks the main thread.

## Configure Serialization

```kotlin
fun Application.configureSerialization() {
    install(ContentNegotiation) {
        json(Json {
            prettyPrint = false
            isLenient = true
            ignoreUnknownKeys = true
        })
    }
}
```

`ContentNegotiation` automatically serializes response objects to JSON and deserializes request bodies from JSON. You write `call.respond(user)` and Ktor converts it to JSON. You write `call.receive<CreateUserRequest>()` and Ktor parses the JSON body.

## Configure Error Handling

```kotlin
fun Application.configureStatusPages() {
    install(StatusPages) {
        exception<IllegalArgumentException> { call, cause ->
            call.respond(
                HttpStatusCode.BadRequest,
                mapOf("error" to (cause.message ?: "Bad request"))
            )
        }

        exception<NoSuchElementException> { call, cause ->
            call.respond(
                HttpStatusCode.NotFound,
                mapOf("error" to (cause.message ?: "Not found"))
            )
        }

        status(HttpStatusCode.NotFound) { call, status ->
            call.respond(
                status,
                mapOf("error" to "Resource not found")
            )
        }
    }
}
```

`StatusPages` intercepts exceptions and HTTP status codes. It converts them to consistent JSON error responses. Unhandled exceptions become 500 errors automatically.

## Configure Routing

```kotlin
fun Application.configureRouting() {
    val repository = UserRepository()

    routing {
        route("/users") {
            get {
                val users = repository.findAll()
                call.respond(users)
            }

            get("/{id}") {
                val id = call.parameters["id"]
                    ?: return@get call.respond(
                        HttpStatusCode.BadRequest,
                        mapOf("error" to "Missing id")
                    )
                val user = repository.findById(id)
                    ?: return@get call.respond(
                        HttpStatusCode.NotFound,
                        mapOf("error" to "User $id not found")
                    )
                call.respond(user)
            }

            post {
                val request = call.receive<CreateUserRequest>()
                val user = repository.create(request)
                call.respond(HttpStatusCode.Created, user)
            }

            delete("/{id}") {
                val id = call.parameters["id"]
                    ?: return@delete call.respond(
                        HttpStatusCode.BadRequest,
                        mapOf("error" to "Missing id")
                    )
                val deleted = repository.delete(id)
                if (deleted) {
                    call.respond(HttpStatusCode.NoContent)
                } else {
                    call.respond(
                        HttpStatusCode.NotFound,
                        mapOf("error" to "User $id not found")
                    )
                }
            }
        }
    }
}
```

Walk through the POST handler:

1. `call.receive<CreateUserRequest>()` -- Ktor reads the request body and deserializes JSON to a `CreateUserRequest`. If the JSON is invalid, Ktor returns 400 automatically.
2. `repository.create(request)` -- Business logic. Creates the user, assigns an ID.
3. `call.respond(HttpStatusCode.Created, user)` -- Serializes the user to JSON and sends it with a 201 status code.

## Run It

```bash
./gradlew run
```

Test with curl:

```bash
curl -X POST http://localhost:8080/users \
  -H "Content-Type: application/json" \
  -d '{"name":"Alice","email":"alice@example.com"}'

curl http://localhost:8080/users
curl http://localhost:8080/users/{id-from-response}
curl -X DELETE http://localhost:8080/users/{id}
```

## Key Takeaways

- Ktor uses plugins (`install(ContentNegotiation)`) instead of annotations.
- `call.receive<T>()` deserializes the request body. `call.respond()` serializes the response.
- Define routes in `routing { }` blocks. Group by resource, not by HTTP method.
- Configure error handling with `StatusPages`. Return consistent JSON error responses.
- The server is embedded. No external container. Just `embeddedServer(Netty, ...)`.
