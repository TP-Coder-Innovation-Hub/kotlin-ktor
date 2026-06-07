# Authentication

`[Mid]`

## Authentication vs Authorization

- **Authentication:** Who are you? (Verify identity)
- **Authorization:** What are you allowed to do? (Check permissions)

This guide covers authentication: verifying that a request comes from a known user.

## Common Approaches

| Approach | How It Works | Use Case |
|----------|-------------|----------|
| API Key | Client sends a static key in a header | Service-to-service |
| Session | Server stores session, client sends cookie | Traditional web apps |
| JWT | Client sends a signed token in a header | APIs, microservices |

For Kotlin backend APIs, JWT is the standard.

## What is JWT

JWT (JSON Web Token) is a self-contained token with three parts:

```
header.payload.signature
```

- **Header:** Algorithm and token type
- **Payload:** Claims (user ID, roles, expiration)
- **Signature:** Cryptographic signature to verify integrity

The server signs the token with a secret key. The client stores it and sends it with every request in the `Authorization` header:

```
Authorization: Bearer eyJhbGciOiJIUzI1NiJ9...
```

The server validates the signature and reads the claims. No database lookup needed. The token itself contains the information.

## JWT in Ktor

Add the dependency:

```kotlin
dependencies {
    implementation("io.ktor:ktor-server-auth:3.1.0")
    implementation("io.ktor:ktor-server-auth-jwt:3.1.0")
}
```

### Configure JWT

```kotlin
import com.auth0.jwt.JWT
import com.auth0.jwt.algorithms.Algorithm
import io.ktor.server.application.*
import io.ktor.server.auth.*
import io.ktor.server.auth.jwt.*
import java.util.*

object JwtConfig {
    private const val secret = "your-secret-key-min-32-chars-long!!"
    private const val issuer = "kotlin-ktor-api"
    private const val audience = "kotlin-ktor-users"
    private val algorithm = Algorithm.HMAC256(secret)

    fun generateToken(userId: String, role: String): String =
        JWT.create()
            .withAudience(audience)
            .withIssuer(issuer)
            .withClaim("userId", userId)
            .withClaim("role", role)
            .withExpiresAt(Date(System.currentTimeMillis() + 86_400_000)) // 24 hours
            .sign(algorithm)

    fun configureKtor(application: Application) {
        application.install(Authentication) {
            jwt("auth-jwt") {
                verifier(
                    JWT.require(algorithm)
                        .withAudience(audience)
                        .withIssuer(issuer)
                        .build()
                )
                validate { credential ->
                    val userId = credential.payload.getClaim("userId")?.asString()
                    if (userId != null) {
                        JWTPrincipal(credential.payload)
                    } else {
                        null
                    }
                }
                challenge { _, _ ->
                    call.respond(
                        io.ktor.http.HttpStatusCode.Unauthorized,
                        mapOf("error" to "Token is invalid or expired")
                    )
                }
            }
        }
    }
}
```

Step by step:

1. `generateToken` creates a JWT with userId, role, and 24-hour expiration. Signed with HMAC-SHA256.
2. `configureKtor` installs the Authentication plugin with JWT validation.
3. `verifier` configures the JWT verification (audience, issuer, algorithm).
4. `validate` extracts claims from the token. Returns a `JWTPrincipal` if valid, null if not.
5. `challenge` returns 401 Unauthorized when the token is invalid.

### Login Endpoint

```kotlin
post("/auth/login") {
    val request = call.receive<LoginRequest>()

    val user = userRepository.findByEmail(request.email)
        ?: return@post call.respond(
            HttpStatusCode.Unauthorized,
            mapOf("error" to "Invalid credentials")
        )

    if (!verifyPassword(request.password, user.passwordHash)) {
        return@post call.respond(
            HttpStatusCode.Unauthorized,
            mapOf("error" to "Invalid credentials")
        )
    }

    val token = JwtConfig.generateToken(user.id, user.role)
    call.respond(mapOf("token" to token))
}
```

The login endpoint verifies credentials and returns a JWT. The client stores this token and sends it in subsequent requests.

### Protect Routes

```kotlin
routing {
    route("/users") {
        // Public: no authentication required
        post {
            val request = call.receive<CreateUserRequest>()
            val user = userService.create(request)
            call.respond(HttpStatusCode.Created, user)
        }

        // Protected: requires valid JWT
        authenticate("auth-jwt") {
            get {
                val users = userService.findAll()
                call.respond(users)
            }

            get("/{id}") {
                val id = call.parameters["id"]!!
                val user = userService.findById(id)
                    ?: return@get call.respond(HttpStatusCode.NotFound)
                call.respond(user)
            }
        }
    }
}
```

Wrap routes in `authenticate("auth-jwt")` to require a valid token. Unauthenticated requests receive 401.

### Access Claims in Handlers

```kotlin
get("/me") {
    val principal = call.principal<JWTPrincipal>()!!
    val userId = principal.payload.getClaim("userId").asString()
    val role = principal.payload.getClaim("role").asString()

    val user = userService.findById(userId)
        ?: return@get call.respond(HttpStatusCode.NotFound)

    call.respond(user)
}
```

## Password Hashing

Never store plain-text passwords. Use bcrypt:

```kotlin
dependencies {
    implementation("org.mindrot:jbcrypt:0.4")
}
```

```kotlin
import org.mindrot.jbcrypt.BCrypt

fun hashPassword(password: String): String = BCrypt.hashpw(password, BCrypt.gensalt())

fun verifyPassword(password: String, hash: String): Boolean = BCrypt.checkpw(password, hash)
```

## Key Takeaways

- JWT is the standard for API authentication. Stateless, self-contained, no database lookup.
- Use Ktor's `Authentication` plugin with `jwt` provider.
- Sign tokens with a strong secret. Use HMAC-SHA256 or RSA.
- Hash passwords with bcrypt. Never store plain-text passwords.
- Protect routes with `authenticate("auth-jwt")`. Access claims with `call.principal<JWTPrincipal>()`.
