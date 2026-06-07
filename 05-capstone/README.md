# Capstone Project

`[Senior]`

Build a lightweight Kotlin API from scratch. This project integrates everything from the learning path: Kotlin language features, Ktor routing, coroutines, Exposed for database access, serialization, authentication, testing, logging, and deployment.

## Specification

### Application: Task Tracker API

A REST API for managing tasks. Users can create, read, update, and delete tasks. Tasks belong to projects. Users authenticate with JWT.

### Requirements

**Endpoints:**

```
POST   /auth/register          Register a new user
POST   /auth/login             Login, receive JWT

GET    /projects               List user's projects
POST   /projects               Create a project
GET    /projects/{id}          Get project details

GET    /projects/{id}/tasks    List tasks in a project
POST   /projects/{id}/tasks    Create a task
PATCH  /tasks/{id}             Update a task (title, status, assignee)
DELETE /tasks/{id}             Delete a task
```

**Data Models:**

```kotlin
@Serializable
data class User(
    val id: String,
    val email: String,
    val name: String,
    val createdAt: String
)

@Serializable
data class Project(
    val id: String,
    val name: String,
    val description: String?,
    val ownerId: String,
    val createdAt: String
)

@Serializable
data class Task(
    val id: String,
    val projectId: String,
    val title: String,
    val description: String?,
    val status: TaskStatus,
    val assigneeId: String?,
    val createdAt: String,
    val updatedAt: String
)

enum class TaskStatus { TODO, IN_PROGRESS, DONE }
```

**Business Rules:**

- Only the project owner can delete the project.
- Only authenticated users can create tasks.
- Task status transitions: TODO -> IN_PROGRESS -> DONE. No backwards transitions.
- Email addresses must be unique.
- Project names must not be blank.

### Technical Requirements

**Stack:**

- Kotlin 2.x with K2 compiler
- Ktor 3.x (Netty engine)
- Exposed for database access (PostgreSQL)
- kotlinx.serialization for JSON
- JWT authentication
- Koin for dependency injection
- Logback for structured logging

**Database:**

Three tables: `users`, `projects`, `tasks`. Use migrations (Flyway or manual schema creation).

```kotlin
object Users : Table("users") {
    val id = uuid("id").autoGenerate()
    val email = varchar("email", 255).uniqueIndex()
    val name = varchar("name", 255)
    val passwordHash = varchar("password_hash", 255)
    val createdAt = timestamp("created_at")
    override val primaryKey = PrimaryKey(id)
}

object Projects : Table("projects") {
    val id = uuid("id").autoGenerate()
    val name = varchar("name", 255)
    val description = text("description").nullable()
    val ownerId = uuid("owner_id") references Users.id
    val createdAt = timestamp("created_at")
    override val primaryKey = PrimaryKey(id)
}

object Tasks : Table("tasks") {
    val id = uuid("id").autoGenerate()
    val projectId = uuid("project_id") references Projects.id
    val title = varchar("title", 500)
    val description = text("description").nullable()
    val status = varchar("status", 20).default("TODO")
    val assigneeId = uuid("assignee_id").nullable() references Users.id
    val createdAt = timestamp("created_at")
    val updatedAt = timestamp("updated_at")
    override val primaryKey = PrimaryKey(id)
}
```

**Architecture:**

```
Application.kt
  configureRouting()     -> routes/
  configureSerialization()
  configureSecurity()    -> auth/
  configureMonitoring()  -> logging

routes/
  AuthRoutes.kt
  ProjectRoutes.kt
  TaskRoutes.kt

services/
  AuthService.kt
  ProjectService.kt
  TaskService.kt

repository/
  UserRepository.kt
  ProjectRepository.kt
  TaskRepository.kt

models/
  User.kt
  Project.kt
  Task.kt

database/
  Tables.kt
  DatabaseFactory.kt
```

### Testing Requirements

- Unit tests for all services (use MockK)
- Integration tests for all routes (use `testApplication`)
- At least one end-to-end test with a real database (use Testcontainers)
- Test coroutine suspend functions with `runTest`
- Verify error responses (400, 401, 404, 409)
- Run: `./gradlew test`

### Deployment Requirements

- Fat JAR build with Shadow plugin
- Dockerfile with multi-stage build
- Configuration via environment variables (DATABASE_URL, JWT_SECRET, PORT)
- Health endpoint at `/health`
- Structured logging with request IDs

### Stretch Goals

- GraalVM native image build
- Pagination for list endpoints
- Rate limiting plugin
- OpenAPI documentation generation
- WebSocket endpoint for real-time task updates

## Evaluation Criteria

| Criterion | Weight |
|-----------|--------|
| All endpoints work correctly | 30% |
| Tests pass (unit + integration) | 25% |
| Idiomatic Kotlin (data classes, null safety, coroutines) | 20% |
| Clean architecture (separation of concerns) | 15% |
| Deployment configuration works | 10% |

## Getting Started

```bash
mkdir task-tracker-api && cd task-tracker-api
gradle init
# Select: Application, Kotlin, Kotlin DSL
```

Add Ktor, Exposed, and serialization dependencies. Build incrementally:

1. Define data models and tables.
2. Set up the database connection.
3. Build the repository layer.
4. Build the service layer.
5. Add routes (start with projects, then tasks, then auth).
6. Add JWT authentication.
7. Write tests.
8. Add logging.
9. Configure deployment (fat JAR, Docker).
10. Test the full flow end-to-end.

Build one feature at a time. Run the server after each step. Verify with curl.
