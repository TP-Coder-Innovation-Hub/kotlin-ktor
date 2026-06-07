# Control Flow

`[Entry]`

## if / else

`if` in Kotlin is an expression. It returns a value.

```kotlin
val max = if (a > b) a else b
```

For multi-line branches, the last expression in each block is the return value:

```kotlin
val category = if (score >= 90) {
    println("Excellent")
    "A"
} else if (score >= 80) {
    "B"
} else {
    "C"
}
```

Use `if` for simple two-way decisions. Use `when` for everything else.

## when

`when` replaces switch statements. It is an expression. It returns a value. The compiler checks for exhaustiveness.

```kotlin
fun describe(httpStatus: Int): String = when (httpStatus) {
    200 -> "OK"
    201 -> "Created"
    400 -> "Bad Request"
    401 -> "Unauthorized"
    404 -> "Not Found"
    500 -> "Internal Server Error"
    else -> "Unknown"
}
```

Multiple values on one branch:

```kotlin
when (day) {
    "Saturday", "Sunday" -> "Weekend"
    in listOf("Monday", "Tuesday", "Wednesday", "Thursday", "Friday") -> "Weekday"
    else -> "Invalid"
}
```

Range checks:

```kotlin
when (age) {
    in 0..17 -> "Minor"
    in 18..64 -> "Adult"
    in 65..120 -> "Senior"
    else -> "Invalid age"
}
```

Type checks:

```kotlin
when (value) {
    is String -> "String of length ${value.length}"
    is Int -> "Integer: $value"
    is List<*> -> "List with ${value.size} elements"
    else -> "Unknown type"
}
```

`when` without a subject (replaces if/else chains):

```kotlin
when {
    user.isAdmin -> "Full access"
    user.isActive -> "Limited access"
    else -> "No access"
}
```

Use `when` as your default branching construct. It is more readable than chained `if/else` and the compiler catches missing cases with sealed classes.

## for

Iterate over anything that provides an iterator:

```kotlin
val fruits = listOf("apple", "banana", "cherry")

for (fruit in fruits) {
    println(fruit)
}
```

With indices:

```kotlin
for ((index, fruit) in fruits.withIndex()) {
    println("$index: $fruit")
}
```

Ranges:

```kotlin
for (i in 1..10) {
    println(i)  // 1, 2, 3, ..., 10 (inclusive)
}

for (i in 1 until 10) {
    println(i)  // 1, 2, 3, ..., 9 (exclusive upper bound)
}

for (i in 10 downTo 1 step 2) {
    println(i)  // 10, 8, 6, 4, 2
}
```

## while

```kotlin
var retries = 0
while (retries < 3) {
    if (connect()) break
    retries++
}
```

```kotlin
do {
    val input = readInput()
} while (input.isBlank())
```

`while` and `do-while` are available but less common in Kotlin. Prefer `for` for collection iteration and functional operators (`map`, `filter`, `forEach`) for transformations.

## Expression vs Statement

This distinction matters in Kotlin:

- **Statement:** Performs an action. No value. (`for`, `while`)
- **Expression:** Produces a value. Can be assigned. (`if`, `when`)

```kotlin
// Expression: assigns the result
val result = when (x) {
    1 -> "one"
    else -> "other"
}

// Statement: performs an action, no value
for (i in 1..10) {
    println(i)
}
```

Kotlin biases toward expressions. When you can return a value from a construct, do it. It reduces mutable variables.

## Key Takeaways

- `when` is Kotlin's switch. It is an expression, exhaustive with sealed classes, and supports conditions, types, and ranges.
- `if` is an expression too. Use it for simple two-way branches.
- Prefer functional operators (`map`, `filter`) over `for` loops for collection transformations.
- Understand the expression vs statement distinction. Kotlin favors expressions.
