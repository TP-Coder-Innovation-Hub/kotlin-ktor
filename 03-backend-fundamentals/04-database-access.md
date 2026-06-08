# Database Access with Exposed



## What is Exposed

Exposed is JetBrains' type-safe SQL DSL for Kotlin. You write Kotlin code that the library translates to SQL. The compiler catches query errors at compile time, not at runtime.

No string interpolation. No raw SQL in your business logic.

## Setup

Add Exposed dependencies:

```kotlin
dependencies {
    implementation("org.jetbrains.exposed:exposed-core:0.58.0")
    implementation("org.jetbrains.exposed:exposed-jdbc:0.58.0")
    implementation("org.jetbrains.exposed:exposed-java-time:0.58.0")
    implementation("com.zaxxer:HikariCP:6.2.1")
    runtimeOnly("org.postgresql:postgresql:42.7.4")
}
```

HikariCP manages connection pooling. PostgreSQL is the database driver.

## Define Tables

Tables are singleton objects extending `Table`:

```kotlin
package com.example.database

import org.jetbrains.exposed.sql.Table
import org.jetbrains.exposed.sql.javatime.timestamp

object Users : Table("users") {
    val id = uuid("id").autoGenerate()
    val name = varchar("name", 255)
    val email = varchar("email", 255).uniqueIndex()
    val active = bool("active").default(true)
    val createdAt = timestamp("created_at").defaultExpression(org.jetbrains.exposed.sql.CurrentTimestamp)

    override val primaryKey = PrimaryKey(id)
}
```

Step by step:

- `uuid("id").autoGenerate()` -- UUID column, auto-generated on insert.
- `varchar("name", 255)` -- String column with max length 255.
- `uniqueIndex()` -- Adds a unique constraint on the email column.
- `bool("active").default(true)` -- Boolean column, defaults to true.
- `timestamp("created_at")` -- Timestamp column.
- `primaryKey = PrimaryKey(id)` -- Declares the primary key.

Column types map directly to SQL types. If you write `varchar("email", 255)`, the generated SQL is `email VARCHAR(255)`. If you mistype a column name, the compiler catches it.

## Database Connection

Configure the connection pool:

```kotlin
package com.example.database

import com.zaxxer.hikari.HikariConfig
import com.zaxxer.hikari.HikariDataSource
import org.jetbrains.exposed.sql.Database
import org.jetbrains.exposed.sql.SchemaUtils
import org.jetbrains.exposed.sql.transactions.transaction

fun initDatabase() {
    val config = HikariConfig().apply {
        jdbcUrl = "jdbc:postgresql://localhost:5432/mydb"
        driverClassName = "org.postgresql.Driver"
        username = "postgres"
        password = "postgres"
        maximumPoolSize = 10
    }

    Database.connect(HikariDataSource(config))

    transaction {
        SchemaUtils.create(Users)
    }
}
```

`SchemaUtils.create(Users)` creates the table if it does not exist. In production, use a migration tool (Flyway, Liquibase) instead.

## Repository with Coroutines

Wrap database operations in `newSuspendedTransaction` so they run on the IO dispatcher without blocking request threads:

```kotlin
package com.example.repository

import com.example.database.Users
import com.example.models.User
import kotlinx.coroutines.Dispatchers
import org.jetbrains.exposed.sql.*
import org.jetbrains.exposed.sql.transactions.experimental.newSuspendedTransaction
import java.util.UUID

class UserRepository {

    suspend fun findAll(): List<User> = dbQuery {
        Users.selectAll().map { it.toUser() }
    }

    suspend fun findById(id: UUID): User? = dbQuery {
        Users.selectAll().where { Users.id eq id }
            .map { it.toUser() }
            .singleOrNull()
    }

    suspend fun findByEmail(email: String): User? = dbQuery {
        Users.selectAll().where { Users.email eq email }
            .map { it.toUser() }
            .singleOrNull()
    }

    suspend fun create(name: String, email: String): User = dbQuery {
        val id = Users.insert {
            it[Users.name] = name
            it[Users.email] = email
        } get Users.id

        User(id.toString(), name, email, true)
    }

    suspend fun update(id: UUID, name: String?, email: String?): Boolean = dbQuery {
        Users.update({ Users.id eq id }) {
            name?.let { n -> it[Users.name] = n }
            email?.let { e -> it[Users.email] = e }
        } > 0
    }

    suspend fun delete(id: UUID): Boolean = dbQuery {
        Users.deleteWhere { Users.id eq id } > 0
    }

    private suspend fun <T> dbQuery(block: suspend () -> T): T =
        newSuspendedTransaction(Dispatchers.IO) { block() }
}
```

Step by step for `findById`:

1. `dbQuery { }` wraps the operation in a suspended transaction on `Dispatchers.IO`.
2. `Users.selectAll()` generates `SELECT * FROM users`.
3. `.where { Users.id eq id }` adds `WHERE id = ?` with the parameter bound safely.
4. `.map { it.toUser() }` converts each `ResultRow` to a domain `User` object.
5. `.singleOrNull()` returns the single result or null if not found.

## Map ResultRow to Domain Object

```kotlin
private fun ResultRow.toUser() = User(
    id = this[Users.id].toString(),
    name = this[Users.name],
    email = this[Users.email],
    active = this[Users.active]
)
```

Extension functions on `ResultRow` keep the mapping logic clean and reusable.

## Query Examples

```kotlin
Users.selectAll().where { Users.active eq true }

Users.selectAll().where { Users.name like "%alice%" }

Users.selectAll().where { Users.createdAt greaterEq LocalDateTime.of(2026, 1, 1, 0, 0) }

Users.select(Users.name, Users.email)
    .where { Users.active eq true }
    .orderBy(Users.name)
    .limit(20)
```

Each of these generates type-safe SQL. Column names and types are checked at compile time.

## Key Takeaways

- Exposed is a type-safe SQL DSL. No string interpolation. Compile-time query validation.
- Define tables as `object : Table("name")` singletons with typed columns.
- Use `newSuspendedTransaction(Dispatchers.IO)` for coroutine-safe database access.
- Map `ResultRow` to domain objects with extension functions.
- Use HikariCP for connection pooling. Use Flyway or Liquibase for migrations in production.
