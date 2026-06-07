# Serialization

`[Mid]`

## What is Serialization

Serialization converts objects to a format that can be stored or transmitted (JSON, protobuf, etc.). Deserialization converts it back. In a backend service, every API request and response involves serialization.

## kotlinx.serialization

Kotlin's official serialization library. It works at compile time via a compiler plugin. No runtime reflection. This matters for:

- Performance (no reflection overhead)
- GraalVM native images (reflection requires explicit configuration)
- Tree shaking (unused serializers are removed)

## Setup

Add the serialization plugin to `build.gradle.kts`:

```kotlin
plugins {
    kotlin("jvm") version "2.1.0"
    kotlin("plugin.serialization") version "2.1.0"
}

dependencies {
    implementation("org.jetbrains.kotlinx:kotlinx-serialization-json:1.8.0")
}
```

The `kotlin("plugin.serialization")` plugin generates serializer code at compile time.

## Basic Usage

Annotate classes with `@Serializable`:

```kotlin
import kotlinx.serialization.Serializable
import kotlinx.serialization.encodeToString
import kotlinx.serialization.decodeFromString
import kotlinx.serialization.json.Json

@Serializable
data class User(
    val id: String,
    val name: String,
    val email: String
)
```

Serialize to JSON:

```kotlin
val user = User("1", "Alice", "alice@example.com")
val json = Json.encodeToString(user)
// {"id":"1","name":"Alice","email":"alice@example.com"}
```

Deserialize from JSON:

```kotlin
val input = """{"id":"1","name":"Alice","email":"alice@example.com"}"""
val user = Json.decodeFromString<User>(input)
// User(id=1, name=Alice, email=alice@example.com)
```

## Custom Field Names

Use `@SerialName` when JSON field names differ from Kotlin property names:

```kotlin
@Serializable
data class UserResponse(
    val id: String,
    val name: String,
    @SerialName("email_address")
    val email: String,
    @SerialName("created_at")
    val createdAt: String
)
```

This produces: `{"id":"1","name":"Alice","email_address":"...","created_at":"..."}`

## Optional Fields

Use default values for optional fields:

```kotlin
@Serializable
data class CreateUserRequest(
    val name: String,
    val email: String,
    val role: String = "user",
    val active: Boolean = true
)
```

When the JSON does not include `role` or `active`, defaults are used.

## Nullable Fields

```kotlin
@Serializable
data class Profile(
    val name: String,
    val bio: String? = null,
    val avatarUrl: String? = null
)
```

Nullable properties serialize to null in JSON when the value is null. On deserialization, missing fields use the default value (null in this case).

## Json Configuration

Configure a `Json` instance for your application:

```kotlin
val json = Json {
    ignoreUnknownKeys = true    // Skip unknown fields in input
    prettyPrint = false         // Compact output
    isLenient = true            // Accept non-standard JSON
    encodeDefaults = true       // Include fields with default values in output
}
```

In a Ktor application, configure this once and pass it to the ContentNegotiation plugin:

```kotlin
install(ContentNegotiation) {
    json(Json {
        ignoreUnknownKeys = true
        prettyPrint = false
    })
}
```

With this plugin installed, Ktor automatically serializes response objects and deserializes request bodies:

```kotlin
post("/users") {
    val request = call.receive<CreateUserRequest>()   // Deserialize from request body
    val user = userService.create(request)
    call.respond(HttpStatusCode.Created, user)         // Serialize to response body
}
```

No manual JSON parsing. No Jackson annotations. No reflection.

## Sealed Class Serialization

```kotlin
@Serializable
sealed class Event {
    @Serializable
    @SerialName("user_created")
    data class UserCreated(val userId: String, val name: String) : Event()

    @Serializable
    @SerialName("order_placed")
    data class OrderPlaced(val orderId: String, val total: Double) : Event()
}
```

This serializes with a type discriminator, so deserialization knows which subclass to instantiate.

## Why Not Gson or Jackson

Gson uses reflection. It creates serializers at runtime by inspecting class fields. This is slow and incompatible with GraalVM native images without manual configuration.

Jackson also uses reflection and annotation scanning. It is powerful but heavy for Kotlin backend services.

kotlinx.serialization generates all serialization code at compile time. No reflection. No runtime overhead. It is the standard for Kotlin projects in 2026.

## Key Takeaways

- Use `kotlinx.serialization` with the compiler plugin. Not Gson. Not Jackson.
- Annotate classes with `@Serializable`. Use `@SerialName` for custom JSON field names.
- Use default values for optional fields. Use nullable types for fields that can be null.
- Configure `Json {}` once per application. Pass it to Ktor's ContentNegotiation plugin.
- No reflection. Compile-time code generation. GraalVM-compatible.
