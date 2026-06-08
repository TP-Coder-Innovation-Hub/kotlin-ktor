# Testing



## Testing Strategy

Three levels of tests for a Kotlin backend:

| Level | Scope | Speed | Tool |
|-------|-------|-------|------|
| Unit | Single function or class | Fast (<1ms) | Kotlin test, MockK |
| Integration | Route + serialization + business logic | Moderate (~10ms) | `testApplication` |
| End-to-end | Full server + database | Slow (~1s) | Testcontainers |

Write mostly unit tests. Write integration tests for critical paths. Write end-to-end tests for the most important workflows.

## Unit Testing

Use Kotlin's built-in test assertions or Kotest. This guide uses the built-in `kotlin-test`:

```kotlin
dependencies {
    testImplementation(kotlin("test"))
}
```

Test a service function:

```kotlin
import kotlin.test.Test
import kotlin.test.assertEquals
import kotlin.test.assertTrue

class UserValidatorTest {

    @Test
    fun `valid user passes validation`() {
        val user = CreateUserRequest("Alice", "alice@example.com")
        val result = validate(user)
        assertTrue(result.isValid)
    }

    @Test
    fun `blank name fails validation`() {
        val user = CreateUserRequest("", "alice@example.com")
        val result = validate(user)
        assertEquals("Name must not be blank", result.error)
    }

    @Test
    fun `invalid email fails validation`() {
        val user = CreateUserRequest("Alice", "not-an-email")
        val result = validate(user)
        assertEquals("Invalid email format", result.error)
    }
}
```

Run: `./gradlew test`

## Mocking with MockK

MockK is the standard mocking library for Kotlin. It handles Kotlin-specific features (data classes, companion objects, coroutines).

```kotlin
dependencies {
    testImplementation("io.mockk:mockk:1.13.13")
}
```

```kotlin
import io.mockk.coEvery
import io.mockk.mockk
import kotlinx.coroutines.test.runTest
import kotlin.test.Test
import kotlin.test.assertEquals

class UserServiceTest {
    private val repository = mockk<UserRepository>()
    private val service = UserService(repository)

    @Test
    fun `find existing user`() = runTest {
        coEvery { repository.findById("123") } returns User("123", "Alice", "alice@example.com")

        val result = service.findById("123")

        assertEquals("Alice", result?.name)
    }

    @Test
    fun `find non-existent user returns null`() = runTest {
        coEvery { repository.findById("999") } returns null

        val result = service.findById("999")

        assertEquals(null, result)
    }
}
```

Key points:

- `mockk<UserRepository>()` creates a mock.
- `coEvery { }` stubs a suspend function (the `co` prefix handles coroutines).
- `returns` defines what the mock returns.
- `runTest` provides a coroutine test scope.

## Testing Ktor Routes with testApplication

Ktor provides `testApplication` for integration testing routes without starting a real server:

```kotlin
dependencies {
    testImplementation("io.ktor:ktor-server-test-host:3.1.0")
}
```

```kotlin
import io.ktor.client.request.*
import io.ktor.client.statement.*
import io.ktor.http.*
import io.ktor.server.testing.*
import kotlinx.serialization.json.Json
import kotlin.test.Test
import kotlin.test.assertEquals

class UserRoutesTest {

    @Test
    fun `create user returns 201`() = testApplication {
        application {
            configureSerialization()
            configureRouting(UserRepository())
        }

        val response = client.post("/users") {
            contentType(MediaType.Application.Json)
            setBody("""{"name":"Alice","email":"alice@example.com"}""")
        }

        assertEquals(HttpStatusCode.Created, response.status)
    }

    @Test
    fun `get user returns 200`() = testApplication {
        application {
            configureSerialization()
            configureRouting(UserRepository())
        }

        val createResponse = client.post("/users") {
            contentType(MediaType.Application.Json)
            setBody("""{"name":"Alice","email":"alice@example.com"}""")
        }

        val user = Json.decodeFromString<User>(createResponse.bodyAsText())

        val getResponse = client.get("/users/${user.id}")
        assertEquals(HttpStatusCode.OK, getResponse.status)
    }

    @Test
    fun `get non-existent user returns 404`() = testApplication {
        application {
            configureSerialization()
            configureRouting(UserRepository())
        }

        val response = client.get("/users/nonexistent")
        assertEquals(HttpStatusCode.NotFound, response.status)
    }
}
```

Step by step:

1. `testApplication` creates a test server in memory.
2. `application { }` configures the server (same plugins as production).
3. `client.post("/users")` sends a request to the test server.
4. Assert on the response status and body.

## Coroutine Testing

Use `runTest` from kotlinx-coroutines-test to test suspend functions:

```kotlin
dependencies {
    testImplementation("org.jetbrains.kotlinx:kotlinx-coroutines-test:1.10.0")
}
```

```kotlin
import kotlinx.coroutines.test.runTest
import kotlin.test.Test
import kotlin.test.assertEquals

class CoroutineServiceTest {

    @Test
    fun `concurrent fetch completes`() = runTest {
        val result = service.fetchAll()
        assertEquals(3, result.size)
    }
}
```

`runTest` provides a test dispatcher that skips delays. `delay(1000)` in your code executes instantly in tests. This keeps tests fast.

## Test File Structure

```
src/
  test/kotlin/
    com/example/
      models/
        UserValidatorTest.kt
      services/
        UserServiceTest.kt
      routes/
        UserRoutesTest.kt
      repository/
        UserRepositoryTest.kt
```

Mirror your source structure in the test directory. Each class gets a corresponding test class.

## Key Takeaways

- Write unit tests for business logic. Use MockK for dependencies.
- Write integration tests with `testApplication` for route testing.
- Use `runTest` for coroutine tests. It skips delays automatically.
- Test file structure mirrors source structure.
- Run tests with `./gradlew test`.
