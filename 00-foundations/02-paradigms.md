# Programming Paradigms



## What is a Paradigm

A paradigm is a way of thinking about how code should be organized. It is not a language feature. It is a mental model that shapes how you design solutions.

Three paradigms matter for Kotlin developers:

## Imperative

Tell the computer what to do, step by step. Each statement changes state.

```kotlin
fun sumImperative(numbers: List<Int>): Int {
    var total = 0
    for (number in numbers) {
        total = total + number
    }
    return total
}
```

You manage the state (`total`), you manage the loop, you manage the accumulation. This is direct and readable for simple logic. It becomes noisy when logic grows.

## Object-Oriented (OOP)

Organize code around objects that combine data and behavior. Objects communicate through methods.

```kotlin
class BankAccount(private var balance: Int) {
    fun deposit(amount: Int) {
        require(amount > 0)
        balance += amount
    }

    fun withdraw(amount: Int): Boolean {
        if (amount > balance) return false
        balance -= amount
        return true
    }

    fun getBalance(): Int = balance
}

fun main() {
    val account = BankAccount(100)
    account.deposit(50)
    println(account.getBalance()) // 150
}
```

The `BankAccount` object owns its data (`balance`) and exposes methods to interact with it. External code cannot directly modify the balance. This encapsulation is the core idea of OOP.

Key OOP concepts:

- **Encapsulation:** Hide internal state, expose behavior through methods.
- **Inheritance:** Share code between related types via class hierarchies.
- **Polymorphism:** Treat different types uniformly through shared interfaces.

## Functional

Treat computation as evaluating functions. Avoid mutable state. Compose small functions into larger ones.

```kotlin
fun sumFunctional(numbers: List<Int>): Int = numbers.reduce { acc, n -> acc + n }
```

No mutable variable. No loop. The `reduce` function handles accumulation. You describe *what* you want (the sum), not *how* to compute it step by step.

Functional patterns in Kotlin:

```kotlin
val scores = listOf(85, 42, 91, 67, 73)

val passing = scores.filter { it >= 70 }
// [85, 91, 73]

val doubled = scores.map { it * 2 }
// [170, 84, 182, 134, 146]

val total = scores.reduce { acc, n -> acc + n }
// 358
```

`filter`, `map`, and `reduce` are higher-order functions. They take other functions as arguments (`{ it >= 70 }` is a lambda). This style is declarative: you say what to do, not how to iterate.

## Kotlin is Multi-Paradigm

Kotlin does not force you into one paradigm. You use all three:

| Situation | Paradigm | Example |
|-----------|----------|---------|
| Domain entities | OOP | `data class User(val id: String, val name: String)` |
| Data transformations | Functional | `users.filter { it.active }.map { it.name }` |
| Procedural logic | Imperative | `for (item in items) process(item)` |
| Restricted hierarchies | Functional + OOP | `sealed class Result<out T>` |

The rule is simple: use the paradigm that makes the code clearest for the problem at hand. Kotlin gives you the tools. Judgment about when to use each is what makes a good developer.
