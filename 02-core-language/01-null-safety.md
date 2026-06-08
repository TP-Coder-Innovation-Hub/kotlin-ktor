# Null Safety



## The Billion Dollar Mistake

Tony Hoare invented the null reference in 1965 for the ALGOL language. He later called it his "billion dollar mistake." Null references have caused more crashes, more bugs, and more production incidents than almost any other language design choice.

In most languages, any variable can be null at any time. You find out at runtime, when the application crashes with a NullPointerException.

Kotlin fixes this at the type level.

## Nullable vs Non-Null Types

In Kotlin, types are non-null by default. A nullable type is marked with `?`.

> 🖼️ **[IMAGE_PLACEHOLDER]** — Kotlin null safety type system String vs String safe call elvis

```kotlin
val name: String = "Kotlin"   // Non-null. Cannot hold null.
// name = null                 // Compile error.

val nullable: String? = null   // Nullable. Can hold null.
nullable = "also valid"
```

The compiler enforces this. You cannot assign null to a non-null type. You cannot call methods on a nullable type without handling the null case first.

```kotlin
val name: String? = fetchName()
println(name.length)    // Compile error: name might be null
```

This is the core value proposition. The compiler catches null errors before the code runs.

## Safe Call Operator: ?.

Access a property or method on a nullable type. Returns null if the receiver is null instead of throwing an exception.

```kotlin
val name: String? = fetchName()
println(name?.length)   // Prints the length, or null if name is null
```

Chain safe calls:

```kotlin
val user: User? = fetchUser(id)
val city = user?.address?.city  // null if user or address is null
```

## Elvis Operator: ?:

Provide a default value when the left side is null.

```kotlin
val name: String? = fetchName()
val display = name ?: "Unknown"   // Use name if non-null, otherwise "Unknown"
```

Common pattern with `let`:

```kotlin
val user: User? = fetchUser(id)
user?.let {
    println("Found user: ${it.name}")
} ?: println("User not found")
```

Throw on null:

```kotlin
val user = fetchUser(id) ?: throw NotFoundException("User $id not found")
```

Early return on null:

```kotlin
fun process(user: User?) {
    val resolved = user ?: return
    println(resolved.name)   // resolved is non-null here
}
```

## Not-Null Assertion: !!

Force unwrap. Throws NullPointerException if the value is null.

```kotlin
val name: String? = fetchName()
println(name!!.length)   // Throws NPE if name is null
```

Avoid `!!`. It exists for interop with Java code and legacy APIs. Every use of `!!` is a compromise. If you find yourself reaching for it, consider:

- Can you make the type non-null?
- Can you use `?:` to provide a default?
- Can you use `?.let` to handle the null case?

## Safe Cast: as?

Returns null if the cast fails instead of throwing ClassCastException.

```kotlin
val value: Any = "hello"
val number: Int? = value as? Int   // null, not an exception
val text: String? = value as? String   // "hello"
```

## Null Safety and Java Interop

Java does not have nullable types. When you call Java code from Kotlin, the compiler cannot determine nullability. It uses "platform types" -- types that might be null.

```kotlin
val name = javaObject.getName()   // Platform type: String!
// Could be null. The compiler does not enforce anything.
```

Treat all Java return values as nullable:

```kotlin
val name: String? = javaObject.getName()   // Explicitly nullable
val safe = name ?: "Unknown"               // Handle the null case
```

## Why This Matters

Null safety eliminates an entire category of runtime crashes. In a backend service handling thousands of requests per second, a NullPointerException in one request should not take down the entire service. Kotlin's type system prevents the NPE from existing in the first place.

The discipline is simple:

1. Non-null by default.
2. Explicit `?` for things that can be null.
3. Handle every nullable with `?.`, `?:`, or `?.let`.
4. Avoid `!!`.

## Key Takeaways

- Types are non-null by default. `String?` is nullable. `String` is not.
- Use `?.` for safe access, `?:` for defaults, `?.let` for null-safe blocks.
- Avoid `!!`. It exists for interop, not for everyday code.
- Treat all Java return values as nullable. The compiler cannot protect you at the interop boundary.
