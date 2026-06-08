# What is Programming



## The Recipe Analogy

A program is a recipe. A recipe has steps, executed in order:

```
1. Boil water
2. Add pasta
3. Wait 8 minutes
4. Drain
5. Serve
```

Each step is an instruction. The computer follows every instruction exactly as written. It does not improvise. It does not skip. If step 2 says "add pasta" and you meant "add rice," you get pasta.

Programming is writing those steps in a language the computer understands.

## Three Things Code Does

Every program, from a calculator app to a cloud platform, does three things:

**1. Takes input.** Data comes from somewhere: a keyboard, a file, a network request, a database.

**2. Processes it.** The code makes decisions, transforms data, stores results. This is the logic.

**3. Produces output.** A result goes somewhere: a screen, a file, a network response, a database row.

```
Input -> Process -> Output
```

A web server takes an HTTP request (input), runs business logic (process), and sends a response (output). A game takes controller input, updates the game state, and renders a frame. The pattern is the same.

## Why It Matters

Understanding this model prevents a common mistake: trying to learn a framework before understanding what code does. Ktor, Spring Boot, React -- these are tools that organize input-process-output at scale. The underlying pattern never changes.

When you read any code, ask three questions:

1. What is the input?
2. What is the processing?
3. What is the output?

If you can answer all three, you understand the code.

## From Recipe to Program

Here is a Kotlin function that follows the same pattern:

```kotlin
fun makeGreeting(name: String): String {
    val greeting = "Hello, $name!"
    return greeting
}
```

Step by step:

1. **Input:** `name: String` -- the function receives a string parameter.
2. **Process:** `val greeting = "Hello, $name!"` -- it builds a new string using string interpolation (`$name` inserts the value).
3. **Output:** `return greeting` -- it returns the result.

You call it:

```kotlin
fun main() {
    val result = makeGreeting("Kotlin")
    println(result) // Hello, Kotlin!
}
```

`main()` is the entry point. Every Kotlin program starts here. `println()` prints to the console. That is your first output.

## Key Takeaways

- Code is instructions. Nothing more, nothing less.
- Input, process, output. Every function follows this pattern.
- Learn the pattern before the framework. Frameworks are just organized patterns.
