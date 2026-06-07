# Logging

`[Mid]`

## Why Logging Matters

In production, you cannot debug with `println`. You need structured logs that:

- Include context (request ID, user ID, timestamp)
- Go to a central location (file, log aggregator, stdout)
- Have severity levels (DEBUG, INFO, WARN, ERROR)
- Do not block request threads

## kotlinx.logging + SLF4J

Kotlin's logging library is `kotlinx-logging`. It wraps SLF4J, which is the standard logging facade on the JVM. You choose the SLF4J implementation (Logback is the most common).

```kotlin
dependencies {
    implementation("io.github.microutils:kotlin-logging-jvm:3.0.5")
    runtimeOnly("ch.qos.logback:logback-classic:1.5.16")
}
```

## Basic Usage

```kotlin
import mu.KotlinLogging

class UserService {
    private val logger = KotlinLogging.logger {}

    fun createUser(request: CreateUserRequest): User {
        logger.info { "Creating user with email: ${request.email}" }
        val user = repository.create(request.name, request.email)
        logger.info { "User created: id=${user.id}" }
        return user
    }

    fun findById(id: String): User? {
        logger.debug { "Looking up user: id=$id" }
        return repository.findById(id).also {
            if (it == null) logger.warn { "User not found: id=$id" }
        }
    }
}
```

Key points:

- `KotlinLogging.logger {}` creates a logger instance.
- Use `{ }` lambda syntax for log messages. The string is only constructed if the log level is enabled. `logger.debug { "expensive: ${compute()}" }` does not call `compute()` if DEBUG is disabled.
- Include relevant context in the message (IDs, key values).

## Log Levels

| Level | When to Use |
|-------|------------|
| ERROR | Failures that need immediate attention. Unhandled exceptions. |
| WARN | Unexpected situations that are recoverable. Missing optional data. |
| INFO | Significant business events. User created, order placed, deployment. |
| DEBUG | Detailed diagnostic info. Request/response bodies, query details. |
| TRACE | Very detailed tracing. Function entry/exit, variable values. |

Use INFO for production. Use DEBUG for development. Use TRACE sparingly.

## Structured Logging

Include structured data in your log messages so log aggregators (ELK, Datadog, CloudWatch) can search and filter:

```kotlin
logger.info { "request_id=${call.callId} method=${call.request.httpMethod.value} path=${call.request.path()} status=${response.status()} duration_ms=${duration} user_id=${userId}" }
```

This format is machine-parseable. Log aggregators can extract fields and build dashboards.

## Ktor Request Logging

Use Ktor's `CallLogging` plugin to log every request:

```kotlin
import io.ktor.server.plugins.calllogging.CallLogging
import io.ktor.server.application.*

fun Application.configureLogging() {
    install(CallLogging) {
        level = org.slf4j.event.Level.INFO
        format { call ->
            val status = call.response.status()
            val method = call.request.httpMethod.value
            val path = call.request.path()
            val duration = call.processingTimeMillis
            "method=$method path=$path status=$status duration_ms=$duration"
        }
    }
}
```

Every request is logged with method, path, status, and duration. No manual logging in every handler.

## Logback Configuration

Create `src/main/resources/logback.xml`:

```xml
<configuration>
    <appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender">
        <encoder>
            <pattern>%d{yyyy-MM-dd'T'HH:mm:ss.SSSZ} [%thread] %-5level %logger{36} - %msg%n</pattern>
        </encoder>
    </appender>

    <logger name="com.example" level="INFO"/>
    <logger name="io.ktor" level="INFO"/>

    <root level="INFO">
        <appender-ref ref="STDOUT"/>
    </root>
</configuration>
```

This outputs structured, timestamped logs to stdout. In container environments, stdout is captured by the container runtime and sent to your log aggregator.

Key settings:

- Set `com.example` to INFO for production.
- Set `io.ktor` to INFO to avoid verbose framework logs.
- In development, change levels to DEBUG.

## Logging Coroutines

Coroutines have their own context. Pass context (request ID, trace ID) through the coroutine context, not through thread-local variables:

```kotlin
data class RequestId(val value: String) : AbstractCoroutineContextElement(RequestId) {
    companion object Key : CoroutineContext.Key<RequestId>
}

suspend fun processRequest(id: String) {
    val requestId = coroutineContext[RequestId]?.value ?: "unknown"
    logger.info { "request_id=$requestid Processing user: id=$id" }
}
```

This ensures the request ID is available in every coroutine in the hierarchy.

## What Not to Log

- Passwords, tokens, API keys, secrets
- Full request/response bodies in production (PII, data volume)
- Health check endpoints (they flood logs)
- Personal data (emails, phone numbers) unless required by your logging policy

## Key Takeaways

- Use `kotlinx-logging` with Logback. Lambda syntax for lazy evaluation.
- Include structured context (request ID, user ID, duration) in every log message.
- Use `CallLogging` for automatic request logging.
- Configure Logback with `logback.xml`. Log to stdout in containers.
- Never log secrets, passwords, or tokens.
