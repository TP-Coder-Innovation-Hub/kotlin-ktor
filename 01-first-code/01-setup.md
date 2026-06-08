# Setup: JDK, IDE, Gradle



## Prerequisites

You need three tools:

1. **JDK** (Java Development Kit) -- Kotlin compiles to JVM bytecode. The JDK provides the compiler and runtime.
2. **IDE** -- IntelliJ IDEA (Community Edition is free) has the best Kotlin support. VS Code works with extensions. Fleet (JetBrains) is an option.
3. **Gradle** -- Build tool. Handles dependencies, compilation, and packaging. Uses Kotlin DSL.

## Step 1: Install the JDK

Install JDK 21 (LTS). Use your package manager:

```bash
# macOS (Homebrew)
brew install openjdk@21

# Linux (SDKMAN)
sdk install java 21.0.4-tem

# Verify
java -version
# openjdk version "21.0.4"
```

Set `JAVA_HOME` to point to the JDK installation. Gradle and Kotlin depend on this.

## Step 2: Install an IDE

**IntelliJ IDEA (recommended):**

Download from jetbrains.com/idea. Community Edition is free and sufficient.

- Kotlin support is built-in (JetBrains created the language)
- Code completion, refactoring, debugging all work out of the box
- Gradle Kotlin DSL support is native

**VS Code (alternative):**

Install the Kotlin extension and the Gradle for Java extension. Functional but less polished than IntelliJ for Kotlin.

## Step 3: Create a Kotlin Project with Gradle

Use the Gradle init task:

```bash
mkdir my-first-kotlin && cd my-first-kotlin
gradle init
```

Select:
- Type: **Application**
- Language: **Kotlin**
- Build script: **Kotlin** (this gives you `build.gradle.kts`)

This generates a project with a `main()` function.

## Step 4: Understand the Build File

Open `build.gradle.kts`:

```kotlin
plugins {
    kotlin("jvm") version "2.1.0"
    application
}

group = "com.example"
version = "1.0.0"

repositories {
    mavenCentral()
}

dependencies {
    testImplementation(kotlin("test"))
}

application {
    mainClass.set("MainKt")
}
```

Key parts:

- `plugins { kotlin("jvm") }` -- the Kotlin JVM compiler plugin
- `repositories { mavenCentral() }` -- where to download dependencies
- `dependencies { }` -- libraries your project uses
- `mainClass.set("MainKt")` -- the entry point (file name + `Kt` suffix)

## Step 5: Write and Run Your First Program

Open `src/main/kotlin/Main.kt`:

```kotlin
fun main() {
    println("Hello, Kotlin!")
}
```

Run it:

```bash
./gradlew run
# > Hello, Kotlin!
```

`gradlew` is the Gradle Wrapper. It downloads the correct Gradle version automatically. Always use `./gradlew`, not `gradle` directly.

## Step 6: Run Tests

The generated project includes a test file at `src/test/kotlin/MainTest.kt`:

```kotlin
import kotlin.test.Test
import kotlin.test.assertEquals

class MainTest {
    @Test
    fun testGreeting() {
        assertEquals("Hello, Kotlin!", createGreeting("Kotlin"))
    }
}
```

Run tests:

```bash
./gradlew test
```

## Project Structure

```
my-first-kotlin/
  build.gradle.kts        # Build configuration
  settings.gradle.kts     # Project name and modules
  gradle/
    libs.versions.toml    # Version catalog (dependency versions)
  src/
    main/kotlin/          # Source code
      Main.kt
    test/kotlin/          # Tests
      MainTest.kt
```

## Key Takeaways

- Install JDK 21, IntelliJ IDEA, and Gradle.
- Use `gradle init` to scaffold a project with Kotlin DSL.
- `./gradlew run` to execute, `./gradlew test` to test.
- The entry point is `fun main()`.
