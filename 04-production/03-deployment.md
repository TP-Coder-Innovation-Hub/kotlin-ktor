# Deployment

`[Senior]`

## Three Deployment Options

| Method | Startup Time | Binary Size | Use Case |
|--------|-------------|-------------|----------|
| Fat JAR | 2-3 seconds | 30-50 MB | Traditional servers, VMs |
| Docker container | 2-3 seconds + pull | 200-400 MB image | Kubernetes, cloud platforms |
| GraalVM native image | 20-50 ms | 50-80 MB | Serverless, CLI, cold-start sensitive |

## Fat JAR with Shadow Plugin

A fat JAR bundles your application and all dependencies into a single executable JAR.

```kotlin
plugins {
    kotlin("jvm") version "2.1.0"
    id("com.github.johnrengelman.shadow") version "8.1.1"
    application
}

application {
    mainClass.set("com.example.ApplicationKt")
}

tasks.build {
    dependsOn(tasks.shadowJar)
}
```

Build:

```bash
./gradlew shadowJar
```

Output: `build/libs/my-application-1.0.0-all.jar`

Run:

```bash
java -jar build/libs/my-application-1.0.0-all.jar
```

This is the simplest deployment model. One file. One command. Works anywhere with a JVM.

Production flags:

```bash
java -Xms256m -Xmx512m \
     -XX:+UseZGC \
     -Dfile.encoding=UTF-8 \
     -jar my-application-1.0.0-all.jar
```

- `-Xms256m -Xmx512m` -- Set heap size. Adjust based on your application's needs.
- `-XX:+UseZGC` -- Use the Z Garbage Collector for low-latency applications.
- `-Dfile.encoding=UTF-8` -- Ensure consistent encoding.

## Docker

Create a `Dockerfile`:

```dockerfile
FROM eclipse-temurin:21-jre-alpine

WORKDIR /app

COPY build/libs/my-application-1.0.0-all.jar app.jar

EXPOSE 8080

ENTRYPOINT ["java", "-Xms256m", "-Xmx512m", "-XX:+UseZGC", "-jar", "app.jar"]
```

Step by step:

1. `eclipse-temurin:21-jre-alpine` -- Minimal JRE image. Alpine-based for small size.
2. `COPY` -- Copy the fat JAR into the container.
3. `EXPOSE 8080` -- Document the port (informational, does not publish).
4. `ENTRYPOINT` -- Run the JAR with production JVM flags.

Build and run:

```bash
docker build -t my-kotlin-api .
docker run -p 8080:8080 my-kotlin-api
```

Multi-stage build for smaller images:

```dockerfile
FROM gradle:8.12-jdk21 AS build
WORKDIR /app
COPY . .
RUN gradle shadowJar --no-daemon

FROM eclipse-temurin:21-jre-alpine
WORKDIR /app
COPY --from=build /app/build/libs/*-all.jar app.jar
EXPOSE 8080
ENTRYPOINT ["java", "-Xms256m", "-Xmx512m", "-XX:+UseZGC", "-jar", "app.jar"]
```

The first stage builds the JAR. The second stage copies only the JAR into a minimal runtime image. The final image does not contain the build tools or source code.

## GraalVM Native Image

Compile Kotlin to a standalone native binary. No JVM required. Startup in 20-50ms.

```kotlin
plugins {
    kotlin("jvm") version "2.1.0"
    id("io.ktor.plugin") version "3.1.0"
}

application {
    mainClass.set("com.example.ApplicationKt")
}

ktor {
    fatJar {
        archiveFileName.set("my-kotlin-api.jar")
    }
    docker {
        jreVersion.set(JavaVersion.VERSION_21)
        localImageName.set("my-kotlin-api")
    }
    nativeImage {
        imageName.set("my-kotlin-api")
    }
}
```

Build:

```bash
./gradlew nativeImage
```

This requires GraalVM installed on the build machine. The output is a native binary:

```bash
./build/native/nativeCompile/my-kotlin-api
```

Trade-offs:

- Fast startup (20-50ms). Ideal for serverless and CLI tools.
- Lower memory footprint (no JVM overhead).
- Longer build time (2-5 minutes for native compilation).
- Limited reflection. kotlinx.serialization works because it is compile-time. Jackson and Gson require explicit reflection configuration.
- Reduced JIT optimization over time. The JVM's JIT compiler optimizes hot code during runtime. Native images are optimized at build time only.

When to choose native image:

- Serverless functions where cold starts matter
- CLI tools where instant startup is expected
- Resource-constrained environments

When to stick with JVM:

- Long-running services where JIT optimization outweighs startup cost
- Applications that depend on runtime reflection
- Teams that cannot afford the longer build times

## Health Checks

Add a health endpoint for orchestrators (Kubernetes, AWS ECS):

```kotlin
routing {
    get("/health") {
        val dbHealthy = try {
            databasehealthcheck()
            true
        } catch (_: Exception) {
            false
        }

        if (dbHealthy) {
            call.respondText("OK", status = HttpStatusCode.OK)
        } else {
            call.respondText("Unhealthy", status = HttpStatusCode.ServiceUnavailable)
        }
    }
}
```

Orchestrators call `/health` periodically. If it returns non-200, the container is restarted.

## Environment Configuration

Read configuration from environment variables, not hardcoded values:

```kotlin
val port = System.getenv("PORT")?.toIntOrNull() ?: 8080
val dbUrl = System.getenv("DATABASE_URL") ?: "jdbc:postgresql://localhost:5432/mydb"
val dbUser = System.getenv("DATABASE_USER") ?: "postgres"
val dbPassword = System.getenv("DATABASE_PASSWORD") ?: error("DATABASE_PASSWORD required")
val jwtSecret = System.getenv("JWT_SECRET") ?: error("JWT_SECRET required")
```

Never hardcode secrets in source code. Never commit secrets to version control. Use environment variables or secret managers (AWS Secrets Manager, HashiCorp Vault).

## Key Takeaways

- Fat JAR: simplest deployment. One file, one command.
- Docker: standard for Kubernetes and cloud platforms. Use multi-stage builds.
- GraalVM native image: fastest startup. Trade-offs in build time and reflection support.
- Add a `/health` endpoint for orchestrators.
- Read all configuration from environment variables. Never hardcode secrets.
