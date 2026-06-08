# Functions



## Basic Functions

```kotlin
fun add(a: Int, b: Int): Int {
    return a + b
}
```

Parts: `fun` keyword, function name, parameters with types, return type, body.

## Single-Expression Functions

When the body is a single expression, use `=`:

```kotlin
fun add(a: Int, b: Int): Int = a + b
```

The return type can be inferred:

```kotlin
fun add(a: Int, b: Int) = a + b
```

Use single-expression functions for simple logic. Use block bodies for anything with multiple steps.

## Default Parameters

Set default values directly in the function signature:

```kotlin
fun greet(name: String, greeting: String = "Hello"): String {
    return "$greeting, $name!"
}

greet("Kotlin")              // "Hello, Kotlin!"
greet("Kotlin", "Welcome")   // "Welcome, Kotlin!"
```

This replaces method overloading. In Java, you would write three versions of the same method. In Kotlin, one function with defaults.

## Named Arguments

Call functions with parameter names for clarity:

```kotlin
fun createUser(name: String, email: String, role: String = "user", active: Boolean = true): User

createUser(
    name = "Alice",
    email = "alice@example.com",
    active = false
)
```

Named arguments make calls with multiple parameters readable. You can skip middle defaults (`role` uses its default) and specify only what matters.

## Extension Functions

This is Kotlin's identity feature. Add methods to existing classes without modifying them.

```kotlin
fun String.isEmail(): Boolean = this.contains("@") && this.contains(".")

fun Int.isEven(): Boolean = this % 2 == 0

fun List<Int>.sumPositive(): Int = this.filter { it > 0 }.sum()
```

Usage:

```kotlin
"test@example.com".isEmail()  // true
42.isEven()                    // true
listOf(-1, 2, 3).sumPositive() // 5
```

Inside an extension function, `this` refers to the receiver (the object you are extending). You can omit `this`:

```kotlin
fun String.shout(): String = uppercase() + "!"
```

When to use extensions:

- Utility methods on standard library types (`String`, `List`, `Int`)
- Adapter patterns (converting one type to another)
- Adding domain-specific methods to library types

When not to use extensions:

- Core domain logic (put it on the class itself)
- Methods that need access to private members (extensions cannot see them)

## Higher-Order Functions

Functions can take other functions as parameters or return them:

```kotlin
fun operate(a: Int, b: Int, operation: (Int, Int) -> Int): Int {
    return operation(a, b)
}

operate(3, 4) { x, y -> x + y }  // 7
operate(3, 4) { x, y -> x * y }  // 12
```

The last parameter is a lambda (`{ x, y -> x + y }`). When the last parameter is a function, Kotlin allows trailing lambda syntax: you write the lambda outside the parentheses.

## Function Types

```kotlin
val doubler: (Int) -> Int = { x -> x * 2 }
val greeter: (String) -> String = { "Hello, $it" }

doubler(5)    // 10
greeter("Kotlin")  // "Hello, Kotlin"
```

`it` is the implicit name for a single lambda parameter.

## Scope Functions

Kotlin provides `let`, `also`, `apply`, `run`, and `with` for operating on objects within a scope. The two most common:

```kotlin
val user: User? = fetchUser(id)

user?.let {
    // Executes only if user is not null
    println(it.name)
}
```

```kotlin
val config = StringBuilder().apply {
    append("host=localhost\n")
    append("port=5432\n")
}
```

Use them when they improve readability. Do not force them where a simple variable assignment is clearer.

## Key Takeaways

- Use single-expression functions (`=`) for simple logic.
- Default parameters replace method overloading.
- Named arguments make multi-parameter calls readable.
- Extension functions add methods to existing classes. This is Kotlin's signature feature.
- Higher-order functions and lambdas are the foundation for functional collection operations.
