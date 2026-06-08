# Classes and Objects



## Basic Class

```kotlin
class Person(val name: String, val age: Int)
```

`val` in the constructor creates a read-only property. `var` creates a mutable one.

```kotlin
val person = Person("Alice", 30)
println(person.name)  // Alice
```

## data class

The most common class type in Kotlin. Automatically generates `equals()`, `hashCode()`, `toString()`, `copy()`, and destructuring.

```kotlin
data class User(val id: String, val name: String, val email: String)

val user1 = User("1", "Alice", "alice@example.com")
val user2 = User("1", "Alice", "alice@example.com")

user1 == user2                  // true (structural equality via equals())
println(user1)                  // User(id=1, name=Alice, email=alice@example.com)

val updated = user1.copy(email = "new@example.com")  // copy with changes
```

When to use data classes:

- DTOs (data transfer objects) for API requests and responses
- Domain entities where equality is based on content
- Any class that primarily holds data

Rules:

- The primary constructor must have at least one parameter.
- All primary constructor parameters must be `val` or `var`.
- Data classes cannot be abstract, open, sealed, or inner.

## sealed class

A restricted class hierarchy. The compiler knows all subclasses at compile time. This enables exhaustive `when` expressions.

```kotlin
sealed class NetworkResult<out T> {
    data class Success<T>(val data: T) : NetworkResult<T>()
    data class Error(val code: Int, val message: String) : NetworkResult<Nothing>()
    data object Loading : NetworkResult<Nothing>()
}

fun handle(result: NetworkResult<String>) = when (result) {
    is NetworkResult.Success -> println("Got: ${result.data}")
    is NetworkResult.Error  -> println("Failed: ${result.message}")
    is NetworkResult.Loading -> println("Loading...")
    // No else needed. Compiler knows all cases are covered.
}
```

If you add a new subclass later, the compiler forces you to update every `when` expression that handles the sealed class. This prevents bugs from forgotten cases.

When to use sealed classes:

- Result types (success, error, loading)
- State machines (idle, running, paused, completed)
- Event hierarchies (user events, system events, network events)
- Any restricted set of types where exhaustiveness matters

## object

Kotlin's singleton. One instance, created lazily on first access.

```kotlin
object DatabaseConfig {
    val host = "localhost"
    val port = 5432
    val maxConnections = 10
}

DatabaseConfig.host  // Access directly via the object name
```

Use `object` for:

- Configuration singletons
- Utility holders (though extension functions and top-level functions are usually better)
- Table definitions in Exposed (covered in 03-backend-fundamentals/04-database-access.md)

## companion object

A singleton inside a class. Like static members in Java, but it is a real object.

```kotlin
class User(val name: String, val email: String) {
    companion object {
        fun fromJson(json: String): User {
            val parts = json.split(",")
            return User(parts[0], parts[1])
        }
    }
}

User.fromJson("Alice,alice@example.com")
```

Use companion objects for:

- Factory methods (`fromJson`, `create`, `default`)
- Constants associated with the class
- Alternative constructors

## init Block

Run initialization logic when an instance is created:

```kotlin
data class CreateUserRequest(val name: String, val email: String) {
    init {
        require(name.isNotBlank()) { "Name must not be blank" }
        require(email.contains("@")) { "Invalid email format" }
    }
}
```

`require()` throws `IllegalArgumentException` if the condition is false. Use it for validation at construction time.

## When to Use Each

| Type | Purpose | Example |
|------|---------|---------|
| `class` | General-purpose types | `class Server(val port: Int)` |
| `data class` | Data holders with auto-generated methods | `data class User(val id: String, val name: String)` |
| `sealed class` | Restricted hierarchies, exhaustive handling | `sealed class Result<out T>` |
| `object` | Singletons | `object AppConfig { ... }` |
| `companion object` | Factory methods and class-level members | `User.fromJson(...)` |

## Key Takeaways

- Use `data class` for DTOs and entities. It auto-generates equals, hashCode, toString, and copy.
- Use `sealed class` for restricted hierarchies. The compiler enforces exhaustive `when` branches.
- Use `object` for singletons. Use `companion object` for factory methods and class constants.
- Validate at construction time with `init` blocks and `require()`.
