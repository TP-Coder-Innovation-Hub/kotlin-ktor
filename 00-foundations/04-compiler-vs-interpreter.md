# Compiler vs Interpreter

`[Entry]`

## Two Ways to Run Code

A programming language is a human-readable specification. The computer runs machine code (ones and zeros). Something must translate between the two. There are two approaches:

**Compiled:** Translate the entire program to machine code before running it.

**Interpreted:** Translate and execute one line at a time while running.

## Compiled Languages

You write source code. A compiler translates it to machine code. You run the machine code.

```
Source Code -> [Compiler] -> Machine Code -> [Execute] -> Output
```

Examples: C, C++, Rust, Go.

Trade-offs:

- Fast execution (the machine code is optimized, no translation at runtime)
- Slow compilation (you wait before running)
- Platform-specific (compile for Linux, macOS, Windows separately)
- Errors caught before the program runs

## Interpreted Languages

You write source code. An interpreter reads and executes it line by line.

```
Source Code -> [Interpreter reads and executes] -> Output
```

Examples: Python, Ruby, JavaScript (traditionally).

Trade-offs:

- Fast iteration (run immediately, no compile step)
- Slower execution (translation happens at runtime)
- Platform-agnostic (if the interpreter exists for your platform, the code runs)
- Some errors only appear at runtime

## The Middle Ground: Kotlin and the JVM

Kotlin uses a hybrid approach. The Kotlin compiler translates source code to **JVM bytecode**, not machine code. The **Java Virtual Machine (JVM)** then runs the bytecode.

```
Kotlin Source -> [Kotlin Compiler] -> JVM Bytecode -> [JVM] -> Output
```

### What is the JVM

The JVM is a virtual computer that runs bytecode. It exists for every major operating system. You compile once, run anywhere.

### What the JVM does at runtime:

1. **Class loading:** Loads compiled bytecode into memory.
2. **Bytecode verification:** Checks that the bytecode is safe and well-formed.
3. **Interpretation:** Starts by interpreting bytecode line by line.
4. **JIT compilation:** After code runs frequently enough, the Just-In-Time compiler translates it to optimized machine code for your specific CPU. This is why JVM code gets faster over time ("warm-up").

### Why this matters for Kotlin backend developers:

**Cross-platform:** Compile your Ktor backend once. Deploy the same JAR to Linux, macOS, Windows. The JVM handles platform differences.

**JVM ecosystem:** Kotlin has access to every Java library ever written. Decades of battle-tested code for databases, messaging, monitoring, and more.

**Performance:** The JVM's JIT compiler produces machine code that rivals C++ for long-running processes. The "JVM is slow" myth died a decade ago. Warm-up latency is real but manageable.

**Garbage collection:** The JVM manages memory automatically. Modern GCs (ZGC, Shenandoah) pause for sub-millisecond. You focus on business logic, not manual memory management.

## Kotlin's Other Compilation Targets

Kotlin is not limited to the JVM:

| Target | Output | Use Case |
|--------|--------|----------|
| Kotlin/JVM | JVM bytecode | Backend, Android |
| Kotlin/JS | JavaScript | Web frontend |
| Kotlin/Native | Native binary | iOS, embedded, systems |
| Kotlin/Wasm | WebAssembly | Web (performance-critical) |

For backend development, you target the JVM. It gives you the richest ecosystem and the most mature runtime.

## Key Takeaways

- Kotlin compiles to JVM bytecode, not machine code.
- The JVM runs bytecode on any platform. Write once, run anywhere.
- The JVM's JIT compiler optimizes hot code to near-native speed.
- Kotlin can also compile to JavaScript, native binaries, and WebAssembly. JVM is the backend target.
