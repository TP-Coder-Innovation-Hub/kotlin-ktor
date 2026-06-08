# Collections and Functional Operations



## Collection Types

Kotlin provides two families:

| Type | Read-only | Mutable |
|------|-----------|---------|
| List | `List<T>` | `MutableList<T>` |
| Set | `Set<T>` | `MutableSet<T>` |
| Map | `Map<K, V>` | `MutableMap<K, V>` |

Use the read-only versions by default. Switch to mutable only when you need to modify the collection in place.

```kotlin
val names: List<String> = listOf("Alice", "Bob", "Charlie")
val unique: Set<Int> = setOf(1, 2, 3, 2, 1)       // [1, 2, 3]
val scores: Map<String, Int> = mapOf("Alice" to 95, "Bob" to 87)
```

## map, filter, reduce

The three fundamental functional operations.

**filter** -- keep elements matching a predicate:

```kotlin
val numbers = listOf(1, 2, 3, 4, 5, 6, 7, 8, 9, 10)
val evens = numbers.filter { it % 2 == 0 }
// [2, 4, 6, 8, 10]
```

**map** -- transform each element:

```kotlin
val names = listOf("alice", "bob", "charlie")
val uppercased = names.map { it.uppercase() }
// [ALICE, BOB, CHARLIE]
```

**reduce** -- combine all elements into one:

```kotlin
val numbers = listOf(1, 2, 3, 4, 5)
val sum = numbers.reduce { acc, n -> acc + n }
// 15
```

Chain operations:

```kotlin
val result = users
    .filter { it.active }
    .map { it.name }
    .sorted()
    .take(5)
```

Each operation returns a new collection. The original is unchanged. This is safe for concurrent code because there is no shared mutable state.

## Other Useful Operations

```kotlin
val numbers = listOf(3, 1, 4, 1, 5, 9, 2, 6)

numbers.sorted()                           // [1, 1, 2, 3, 4, 5, 6, 9]
numbers.sortedDescending()                 // [9, 6, 5, 4, 3, 2, 1, 1]
numbers.distinct()                         // [3, 1, 4, 5, 9, 2, 6]
numbers.groupBy { it % 2 }                // {1=[3, 1, 5, 9, 1], 0=[4, 2, 6]}
numbers.associateBy { "key-$it" }         // {key-3=3, key-1=1, key-4=4, ...}
numbers.find { it > 5 }                   // 9 (first match, or null)
numbers.count { it > 3 }                  // 4
numbers.any { it > 10 }                   // false
numbers.all { it > 0 }                    // true
numbers.none { it < 0 }                   // true
numbers.chunked(3)                        // [[3, 1, 4], [1, 5, 9], [2, 6]]
numbers.sum()                             // 31
numbers.average()                         // 3.875
```

## Sequences: Lazy Evaluation

Collection operations (`map`, `filter`) create intermediate collections. For small collections, this is fine. For large collections or multi-step chains, use sequences to evaluate lazily.

```kotlin
val result = numbers
    .asSequence()
    .filter { it > 3 }
    .map { it * 2 }
    .take(3)
    .toList()
```

With a sequence, each element flows through the entire chain one at a time. No intermediate lists are created. The `take(3)` means only the first 3 matching elements are fully processed.

When to use sequences:

- Large collections (thousands+ of elements)
- Multi-step chains where early termination is possible (`take`, `first`, `any`)
- Streaming data from files or databases

When not to use sequences:

- Small collections (overhead of sequence creation is not worth it)
- When you need all results before the next step

## forEach and for

```kotlin
names.forEach { println(it) }

for ((index, name) in names.withIndex()) {
    println("$index: $name")
}
```

Use `forEach` for simple side effects. Use `for` when you need the index or complex iteration logic. Use `map`/`filter` when you are transforming data.

## Destructuring

```kotlin
data class User(val name: String, val age: Int, val email: String)

val user = User("Alice", 30, "alice@example.com")
val (name, age, email) = user

val entries = mapOf("a" to 1, "b" to 2)
for ((key, value) in entries) {
    println("$key = $value")
}
```

## Key Takeaways

- Use `List`, `Set`, `Map` (read-only) by default. Use mutable versions only when needed.
- `filter`, `map`, `reduce` are the core operations. Chain them for data transformations.
- Use sequences for large collections and multi-step chains with early termination.
- Collection operations return new collections. No mutation. Safe for concurrent code.
