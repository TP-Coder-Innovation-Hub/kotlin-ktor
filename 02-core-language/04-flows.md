# Flows



## What is a Flow

A `Flow` is a cold stream of values. Cold means it produces values only when collected. Each collector gets its own stream. Think of it as a sequence that can suspend.

```kotlin
fun countdown(from: Int): Flow<Int> = flow {
    for (i in from downTo 0) {
        emit(i)
        delay(1000)
    }
}
```

Step by step:

1. `flow { }` creates a cold Flow. Nothing runs until someone collects.
2. `emit(i)` sends a value to the collector.
3. `delay(1000)` suspends for one second without blocking a thread.

Collect:

```kotlin
suspend fun main() {
    countdown(5).collect { value ->
        println(value)
    }
}
// Prints 5, 4, 3, 2, 1, 0 with 1-second intervals
```

`collect` is a suspend function. It runs the flow and receives each emitted value.

## Flow Operators

Flows support the same functional operators as collections, plus more:

```kotlin
fun userScores(): Flow<Score> = flow {
    while (currentCoroutineContext().isActive) {
        val scores = scoreService.fetchLatest()
        scores.forEach { emit(it) }
        delay(5000)
    }
}

userScores()
    .filter { it.value > 0 }
    .map { it.copy(value = it.value * 10) }
    .take(20)
    .collect { println(it) }
```

Operators are intermediate. They define transformations. `collect` is terminal -- it triggers execution.

Common operators:

| Operator | Purpose |
|----------|---------|
| `map` | Transform each value |
| `filter` | Keep values matching a predicate |
| `take` | Take first N values, then cancel |
| `drop` | Skip first N values |
| `onEach` | Perform a side effect for each value |
| `onStart` | Perform action when collection starts |
| `onCompletion` | Perform action when collection completes |
| `catch` | Handle upstream exceptions |
| `combine` | Combine with another Flow |
| `flattenMerge` | Flatten a Flow of Flows concurrently |

## Error Handling

```kotlin
fun riskyStream(): Flow<String> = flow {
    emit("first")
    throw RuntimeException("something went wrong")
}

riskyStream()
    .catch { e -> emit("recovered from: ${e.message}") }
    .collect { println(it) }
// Prints: first
// Prints: recovered from: something went wrong
```

`catch` handles exceptions from upstream (before the catch operator). It can emit replacement values.

## StateFlow: Hot Stream with State

> 🖼️ **[IMAGE_PLACEHOLDER]** — Kotlin Flow cold vs hot StateFlow SharedFlow diagram

A `StateFlow` holds a single value and emits updates. It is hot -- it is always active, regardless of collectors. New collectors receive the current value immediately.

```kotlin
class UserViewModel {
    private val _userState = MutableStateFlow<User?>(null)
    val userState: StateFlow<User?> = _userState.asStateFlow()

    fun updateUser(user: User) {
        _userState.value = user
    }
}
```

Usage:

```kotlin
viewModel.userState.collect { user ->
    user?.let { updateUI(it) }
}
```

Use `StateFlow` for state that changes over time and needs to be observed by multiple consumers.

## SharedFlow: Hot Stream for Events

A `SharedFlow` emits values to all active collectors. It does not hold a single state -- it broadcasts events.

```kotlin
class EventBus {
    private val _events = MutableSharedFlow<Event>()
    val events: SharedFlow<Event> = _events.asSharedFlow()

    suspend fun emit(event: Event) {
        _events.emit(event)
    }
}
```

Usage:

```kotlin
eventBus.events.collect { event ->
    handleEvent(event)
}
```

## Flow vs StateFlow vs SharedFlow

| Type | Temperature | Use Case |
|------|-------------|----------|
| `Flow` | Cold | One-shot streams, API responses, transformations |
| `StateFlow` | Hot | State that changes over time (UI state, config) |
| `SharedFlow` | Hot | Events broadcast to multiple consumers |

## Flow vs Channel

| Aspect | Flow | Channel |
|--------|------|---------|
| Elements | Can be replayed | Consumed once |
| Backpressure | Built-in (suspend) | Buffer or suspend |
| Multiple collectors | Each gets own stream | Elements distributed |
| Use case | Streaming data | Producer-consumer queues |

Use Flow for data streams. Use Channel when you need a queue with single-consumption semantics.

## Key Takeaways

- `Flow` is a cold stream. It runs on each `collect`.
- Use operators (`map`, `filter`, `catch`) to transform and handle errors.
- `StateFlow` holds state. New collectors get the current value immediately.
- `SharedFlow` broadcasts events to all active collectors.
- Use Flow for streams. Use suspend functions for single values.
