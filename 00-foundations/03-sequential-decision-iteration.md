# Sequential, Decision, Iteration

`[Entry]`

## The Three Building Blocks

Every program, regardless of language or scale, is built from three constructs:

1. **Sequential:** Do this, then do that.
2. **Decision:** Do this OR do that, based on a condition.
3. **Iteration:** Do this repeatedly until a condition changes.

That is it. Everything else -- functions, classes, frameworks, distributed systems -- is organization layered on top of these three constructs.

## Sequential

Statements execute in order, top to bottom.

```kotlin
fun processOrder(orderId: String) {
    val order = fetchOrder(orderId)    // Step 1
    val validated = validate(order)    // Step 2
    val total = calculateTotal(validated) // Step 3
    saveOrder(total)                   // Step 4
}
```

Each line runs after the previous one completes. If `fetchOrder` fails, the function throws an exception and the remaining lines do not execute.

Sequential execution is the default. You do not need special syntax for it.

## Decision

Branch based on a condition. In Kotlin, the primary decision construct is `when` (and `if` for simple cases).

```kotlin
fun describe(number: Int): String = when {
    number < 0  -> "negative"
    number == 0 -> "zero"
    else        -> "positive"
}
```

`when` is an expression in Kotlin. It returns a value. This is important: Kotlin prefers expressions (which produce values) over statements (which only produce side effects).

```kotlin
val discount = when (customerType) {
    "premium" -> 0.20
    "regular" -> 0.10
    else      -> 0.0
}
```

Simple two-way decisions use `if`:

```kotlin
val label = if (age >= 18) "adult" else "minor"
```

## Iteration

Repeat a block of code. Kotlin provides `for` and `while`.

```kotlin
val items = listOf("a", "b", "c")

for (item in items) {
    println(item)
}
```

```kotlin
var attempts = 0
while (attempts < 3) {
    val success = tryConnect()
    if (success) break
    attempts++
}
```

In practice, Kotlin developers prefer functional iteration over imperative loops:

```kotlin
items.forEach { println(it) }

val uppercased = items.map { it.uppercase() }

val found = items.firstOrNull { it.startsWith("a") }
```

`map`, `filter`, `forEach`, `firstOrNull` -- these handle iteration internally. You provide the logic for each element. The function manages the loop.

## Combining the Three

A real function uses all three:

```kotlin
fun processItems(items: List<String>): List<String> {
    val results = mutableListOf<String>()       // Sequential: declare a variable
    for (item in items) {                       // Iteration: loop over items
        val processed = if (item.length > 3) {  // Decision: branch on condition
            item.uppercase()
        } else {
            item.lowercase()
        }
        results.add(processed)                  // Sequential: add to result
    }
    return results
}
```

Or the same logic, expressed functionally:

```kotlin
fun processItems(items: List<String>): List<String> =
    items.map { item ->
        if (item.length > 3) item.uppercase() else item.lowercase()
    }
```

Both versions do the same thing. The second is preferred in Kotlin because it eliminates mutable state and is more readable.

## Key Takeaways

- Sequential, decision, iteration. Every program uses these three.
- Kotlin `when` replaces switch statements. It is an expression, not a statement.
- Prefer functional iteration (`map`, `filter`) over imperative loops (`for`, `while`) when the logic is simple.
