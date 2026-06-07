# AGENTS.md

Context and guidelines for AI coding assistants working in the Kotlin Backend (Ktor) learning path repository.

## Context

This repository contains educational content for learning backend development with Kotlin and Ktor. It is part of the TP-Coder Innovation Hub learning paths. The audience is software engineers at varying skill levels (`[Entry]`, `[Mid]`, `[Senior]`) who want to build production-grade backend services using Kotlin.

## Audience

- Software engineers learning Kotlin for backend development
- Java developers transitioning to Kotlin
- Engineers evaluating Kotlin/Ktor against other backend stacks (Spring Boot, Go, Node.js)
- All content levels: entry (new to backend or Kotlin), mid (some experience), senior (architectural decisions)

## How to Help

- Write Kotlin code that follows idiomatic conventions (data classes, expression bodies, sealed classes, extension functions where appropriate)
- Use Ktor 3.x APIs and patterns (plugin-based, coroutine-native)
- Use `Exposed` for database access (type-safe SQL DSL, not string-based queries)
- Use `kotlinx.serialization` for JSON serialization (not Gson, not Jackson, unless interop requires it)
- Use Gradle Kotlin DSL for build configuration
- Follow structured concurrency principles (no `GlobalScope`, always use `coroutineScope` or `supervisorScope`)
- Include null-safe patterns in all code examples
- Mark difficulty level with `[Entry]`, `[Mid]`, or `[Senior]` badges where appropriate
- Write clear, concise explanations with code examples before prose
- Use Mermaid diagrams for architectural or flow-based concepts
- Use image placeholders in the format `![Description](./assets/image-name.png)` for conceptual diagrams

## How NOT to Help

- Do not write Spring Boot code or assume Spring annotations (`@Autowired`, `@Service`, etc.)
- Do not use Java-style patterns where Kotlin has idiomatic alternatives (e.g., do not write getters/setters, use data classes)
- Do not use `Gson`, `Jackson`, or reflection-based serialization where `kotlinx.serialization` is the correct choice
- Do not suggest blocking JDBC calls on `Dispatchers.Default`
- Do not use `GlobalScope` or unstructured concurrency
- Do not add comments to every line of code; trust the reader to understand standard Kotlin
- Do not use emojis in code or documentation
- Do not assume the reader knows Android-specific concepts; this is a backend-focused repository
- Do not generate placeholder content; all code examples must be complete and runnable

## Key Concepts

1. **Null safety** -- Types are non-nullable by default (`String` vs `String?`). The compiler enforces null checks. This eliminates NullPointerException at compile time.

2. **Coroutines and structured concurrency** -- Kotlin's concurrency model uses `suspend` functions, `CoroutineScope`, and `Job` hierarchies. Every coroutine has a parent. Cancellation and errors propagate through the hierarchy. `Flow` is the standard for cold streams.

3. **Ktor plugin architecture** -- Ktor is function-based, not annotation-based. Functionality is added through plugins (`install(ContentNegotiation)`, `install(Authentication)`). The request pipeline is explicit and inspectable.

4. **Exposed for database access** -- Type-safe SQL DSL that catches query errors at compile time. Uses `Table` objects to define schemas and lambda-based query builders. No string interpolation for SQL.

5. **kotlinx.serialization** -- Compile-time JSON serialization via a compiler plugin. No runtime reflection. Generates `serializer()` for each `@Serializable` class. Required for GraalVM native image compatibility.

6. **Gradle Kotlin DSL** -- Build scripts are Kotlin code (`build.gradle.kts`). Type-safe, IDE-auto-completed, refactoring-friendly. Use version catalogs (`libs.versions.toml`) for dependency management.

7. **Kotlin Multiplatform (KMP)** -- Allows sharing business logic between backend (JVM), Android, iOS, and web. Not required for backend-only projects, but relevant for organizations with shared logic needs.

## Kotlin/Ktor Guidelines (2026)

### Language: Kotlin 2.x

- Use the K2 compiler frontend (default in Kotlin 2.x)
- Use data classes for DTOs, request/response models, and domain entities
- Use sealed classes/interfaces for result types, state machines, and event hierarchies
- Use expression bodies for single-expression functions
- Use `when` expressions (not statements) for branching logic
- Use `let`, `also`, `apply`, `run`, `with` scope functions appropriately (not excessively)
- Prefer `val` over `var`; prefer immutable collections
- Use extension functions for utility methods, not for core domain logic

### Framework: Ktor 3.x

- Use the Netty engine for production deployments (default)
- Configure plugins in dedicated functions (`configureRouting()`, `configureSerialization()`, etc.)
- Use `call.respond()` and `call.receive<T>()` for request/response handling
- Use `Application` as the configuration entry point
- Use `routing { }` block for route definitions
- Structure routes by domain, not by HTTP method

### Coroutines

- All I/O-bound suspend functions should specify `Dispatchers.IO` via `withContext`
- Use `coroutineScope` for structured concurrency (children fail together)
- Use `supervisorScope` for independent children (one failure does not cancel others)
- Use `Flow` for streaming data; use `suspend` functions for single-value results
- Always handle `CancellationException` correctly (re-throw it, do not swallow it)
- Use `ensureActive()` in CPU-bound loops to support cancellation

### Database: Exposed

- Define tables as `object : Table("table_name")` singletons
- Use `newSuspendedTransaction(Dispatchers.IO)` for database operations
- Map `ResultRow` to domain objects with extension functions
- Use Exposed's DSL (`select`, `insert`, `update`, `delete`) not raw SQL strings
- Use `@Serializable` on response DTOs, not on Exposed table objects

### Serialization: kotlinx.serialization

- Annotate classes with `@Serializable`
- Use `@SerialName` for custom JSON field names
- Configure `Json {}` instance once and inject it via Koin or Application configuration
- Use `encodeToString` / `decodeFromString` for manual serialization
- ContentNegotiation plugin handles request/response serialization automatically

### Dependency Injection: Koin

- Use Koin's module system for defining dependencies
- Use `single` for singleton services, `factory` for new instances per request
- Use `get()` and `inject()` for dependency resolution
- Define modules per feature/domain
- Do not use Koin annotations or code generation; use the manual DSL

### Testing

- Use `testApplication` from `ktor-server-test-host` for integration tests
- Use MockK for mocking Kotlin classes and interfaces
- Test suspend functions directly in `runTest` (from kotlinx-coroutines-test)
- Test routes with `handleRequest` and `handleWebSocketConversation`
- Use Testcontainers for database integration tests

## Repository Structure

```
kotlin-ktor/
  README.md                               # Navigation table and learning objectives
  AGENTS.md                               # This file
  00-foundations/
    01-what-is-programming.md             # What code does
    02-paradigms.md                       # OOP, functional, imperative
    03-sequential-decision-iteration.md   # The 3 building blocks
    04-compiler-vs-interpreter.md         # JVM bytecode and why it matters
    05-what-is-kotlin.md                  # History and motivation
    06-why-kotlin-why-not-x.md            # Trade-offs vs alternatives
  01-first-code/
    01-setup.md                           # JDK, IDE, Gradle setup
    02-variables-and-types.md             # val, var, type inference
    03-control-flow.md                    # if, when, for, while
    04-functions.md                       # Functions and extensions
    05-classes-and-objects.md             # data, sealed, object, companion
  02-core-language/
    01-null-safety.md                     # Nullable types and operators
    02-collections-and-functional.md      # map, filter, sequences
    03-coroutines.md                      # suspend, async, structured concurrency
    04-flows.md                           # Flow, StateFlow, SharedFlow
    05-serialization.md                   # kotlinx.serialization
  03-backend-fundamentals/
    01-http-and-web-servers.md            # HTTP basics
    02-rest-api-design.md                 # REST conventions
    03-your-first-api.md                  # Ktor routing and handlers
    04-database-access.md                 # Exposed type-safe SQL
    05-authentication.md                  # JWT, sessions, auth concepts
  04-production/
    01-testing.md                         # Testing strategies
    02-logging.md                         # Structured logging
    03-deployment.md                      # Fat JAR, Docker, GraalVM
  05-capstone/
    README.md                             # Full project specification
```
