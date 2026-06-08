# Variables and Types



## val vs var

Kotlin has two variable declarations:

- `val` -- read-only (immutable). Assign once. Cannot reassign.
- `var` -- mutable. Can reassign.

```kotlin
val name = "Kotlin"  // read-only
// name = "Java"     // Compile error: Val cannot be reassigned

var counter = 0       // mutable
counter = 1           // OK
```

**Use `val` by default.** Mutable state is the source of most bugs in concurrent code. If you do not need to reassign, do not allow it. Switch to `var` only when you have a specific reason.

This is not a suggestion. It is a convention enforced by code review and linters across the Kotlin ecosystem.

## Type Inference

You usually do not need to write the type. The compiler figures it out.

```kotlin
val name = "Kotlin"        // inferred as String
val year = 2011            // inferred as Int
val price = 29.99          // inferred as Double
val active = true          // inferred as Boolean
```

When the type is ambiguous or you want to be explicit:

```kotlin
val id: String = "abc123"
val timeout: Long = 30_000L
```

## Basic Types

Kotlin does not have primitive types in the language. Everything is an object. The compiler optimizes to primitives at the bytecode level when possible.

| Type | Description | Example |
|------|-------------|---------|
| `String` | Text | `"hello"` |
| `Int` | 32-bit integer | `42` |
| `Long` | 64-bit integer | `9999999999L` |
| `Double` | 64-bit floating point | `3.14` |
| `Float` | 32-bit floating point | `3.14f` |
| `Boolean` | True or false | `true`, `false` |
| `Byte` | 8-bit integer | `127` |
| `Short` | 16-bit integer | `32767` |
| `Char` | Single character | `'A'` |
| `Unit` | No meaningful value (like void) | `Unit` |
| `Nothing` | Function never returns (throws) | `Nothing` |

## Strings

String templates let you embed expressions:

```kotlin
val language = "Kotlin"
val version = 2.1

println("$language version $version")           // Kotlin version 2.1
println("${language.uppercase()} is great")     // KOTLIN is great
```

Multi-line strings use triple quotes:

```kotlin
val query = """
    SELECT u.id, u.name, u.email
    FROM users u
    WHERE u.active = true
    ORDER BY u.name
""".trimIndent()
```

`trimIndent()` removes the leading whitespace so the string is clean.

## Type Conversion

Kotlin does not implicitly convert between numeric types. You must convert explicitly:

```kotlin
val intVal: Int = 42
val longVal: Long = intVal.toLong()     // explicit conversion
val doubleVal: Double = intVal.toDouble()
val stringVal: String = intVal.toString()
```

This prevents accidental precision loss. The compiler will not silently convert an Int to a Double and lose information.

## Why This Matters

The `val`-by-default rule combined with explicit null safety (covered in 02-core-language/01-null-safety.md) eliminates two large classes of bugs:

1. Accidental reassignment causing unexpected state changes
2. Implicit type conversions causing precision loss or overflow

These are constraints, not limitations. They make code predictable.

## Key Takeaways

- Use `val` by default. Use `var` only when you need mutability.
- The compiler infers types. Be explicit when it improves readability.
- Kotlin has no primitive types in the language. The compiler optimizes to primitives at bytecode level.
- Numeric conversions are explicit, never implicit.
