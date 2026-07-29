# Java Runtime and Performance Internals

## Quick Navigation Index

Use this guide as a concise map of how Java code runs, how memory is managed, and where performance issues usually originate.

- [1. Why this matters](#1-why-this-matters)
- [2. The Java execution pipeline](#2-the-java-execution-pipeline)
- [3. The JVM architecture](#3-the-jvm-architecture)
  - [3.1 Class loader subsystem](#31-class-loader-subsystem)
  - [3.2 Runtime data areas](#32-runtime-data-areas)
- [4. Garbage collection in Java 17](#4-garbage-collection-in-java-17)
- [5. The execution engine](#5-the-execution-engine)
- [6. Memory model and object lifecycle](#6-memory-model-and-object-lifecycle)
- [7. Multithreading and concurrency in the JVM](#7-multithreading-and-concurrency-in-the-jvm)
- [8. Class initialization and static behavior](#8-class-initialization-and-static-behavior)
- [9. Performance tuning mindset](#9-performance-tuning-mindset)
- [10. Example runtime behavior](#10-example-runtime-behavior-of-a-simple-program)
- [11. Key takeaways](#11-key-takeaways)

---

## 1. Why this matters

Java is often described as “write once, run anywhere,” but that promise is realized by the Java Virtual Machine (JVM). The JVM is not just a runtime; it is a full execution engine that handles class loading, memory management, code execution, optimization, and concurrency. Understanding the execution engine is essential for writing performant, reliable, and scalable Java applications.

This document covers the end-to-end lifecycle of a Java program from source to execution, including bytecode, class loading, memory areas, garbage collection, JIT compilation, threading, and runtime tuning.

---

## 2. The Java execution pipeline

When you run a Java program, the flow is:

1. Source code is compiled by the Java compiler into bytecode.
2. The JVM loads the bytecode classes into memory.
3. The class loader resolves dependencies and links classes.
4. The JVM interprets or compiles bytecode to native machine code.
5. The runtime executes the code while managing memory and threads.
6. Garbage collection frees unreachable objects.

### 2.1 Source code to bytecode

Java source files are compiled to `.class` files using the Java compiler.

Example:

```java
public class HelloWorld {
    public static void main(String[] args) {
        System.out.println("Hello");
    }
}
```

The compiler produces bytecode that is platform-neutral. That bytecode is what the JVM executes.

### 2.2 Why bytecode matters

Bytecode is a portable instruction set for the JVM. It allows Java to run on any platform that has a compatible JVM implementation. It also creates a layer of abstraction that enables runtime optimizations.

---

## 3. The JVM architecture

The JVM can be conceptualized as having four core subsystems:

- Class loader subsystem
- Runtime data areas
- Execution engine
- Native interface and garbage collector

### 3.1 Class loader subsystem

The class loader loads `.class` files into memory when required. It follows the delegation model.

#### 3.1.1 Built-in class loaders

Java 17 uses the following class loaders:

- Bootstrap class loader: loads core JDK classes like `java.lang`, `java.util`.
- Platform class loader: loads Java SE platform classes and modules.
- Application class loader: loads application classes from the classpath.

#### 3.1.2 Delegation principle

A class loader typically asks its parent first. If the parent cannot find the class, it tries itself.

This provides:

- Better security
- Avoids duplicate class loading
- Allows modular isolation

#### 3.1.3 Class loading phases

A class goes through:

1. Loading
2. Linking
3. Verification
4. Preparation
5. Resolution
6. Initialization

The most important to understand are:

- Loading: locating and reading the bytecode
- Linking: combining class definitions with runtime structures
- Verification: ensuring bytecode is valid and safe
- Initialization: executing static initializers

### 3.2 Runtime data areas

The JVM stores data in several memory regions.

#### 3.2.1 Heap

The heap is where objects are allocated. It is shared by all threads.

The heap is divided into generations:

- Young generation
- Old generation
- Metaspace

#### 3.2.2 Stack

Each thread has its own stack. It stores method frames, local variables, and partial results.

#### 3.2.3 Program Counter Register

Each thread has a program counter that tracks the current instruction address.

#### 3.2.4 Native Method Stack

Used for native methods implemented in languages such as C/C++.

#### 3.2.5 Metaspace

The metaspace stores class metadata, method metadata, and other JVM internal structures. Unlike the old permanent generation, metaspace is native memory-backed.

---

## 4. Garbage collection in Java 17

Garbage collection is the process of reclaiming memory used by objects that are no longer reachable.

### 4.1 Reachability

An object is considered garbage if no live reference chain can reach it.

Examples:

- Objects assigned `null`
- Objects that are no longer referenced by any active path
- Cycles that are not reachable from root objects

### 4.2 GC roots

Garbage collectors begin from GC roots such as:

- Local variables on the stack
- Static fields
- JNI references
- Thread objects

### 4.3 Common collector types

Java 17 supports various garbage collectors, including:

- Serial GC
- Parallel GC
- G1 GC
- ZGC
- Shenandoah

### 4.4 G1 GC (important for modern workloads)

G1 is the default collector in many Java 17 environments. It aims to provide predictable pause times while balancing throughput.

Key ideas:

- Heap is divided into regions
- GC works region by region
- Young and old generation are handled incrementally

### 4.5 Why GC matters

Poor GC tuning can cause:

- Long pause times
- High memory usage
- High CPU consumption
- Application stalls

### 4.6 Tuning considerations

Common tuning flags include:

```bash
-Xms512m -Xmx2g -XX:+UseG1GC
```

Useful tuning principles:

- Set heap size based on workload
- Avoid excessive heap pressure
- Monitor GC pauses and allocation rate
- Use GC logs for diagnosis

Example:

```bash
java -Xms1g -Xmx2g -XX:+UseG1GC -Xlog:gc*:file=gc.log -jar app.jar
```

---

## 5. The execution engine

The execution engine is responsible for executing the bytecode.

### 5.1 Interpreter

The interpreter executes bytecode one instruction at a time. It is simple but slower.

### 5.2 JIT compiler

The Just-In-Time compiler compiles hot code paths to native machine code. This greatly improves performance.

#### 5.2.1 Why JIT exists

The JVM profiles application behavior at runtime. Frequently executed code is compiled into optimized machine code.

#### 5.2.2 Hot methods

Methods that are executed repeatedly become candidates for JIT optimization.

#### 5.2.3 Tiered compilation

Java 17 uses tiered compilation in many default configurations. This means:

- Bytecode starts interpreting
- Hot methods are compiled quickly
- More aggressive optimization occurs later

### 5.3 C1 and C2 compilers

Some JVMs use multiple compilation tiers. The JVM can compile code in a fast, less optimized way first and then move to more optimized strategies as the code stays hot.

---

## 6. Memory model and object lifecycle

### 6.1 Object creation

Objects are created on the heap. References exist in stack frames or in fields.

### 6.2 Escape analysis

The JVM can detect that an object does not escape a method and optimize it away or stack-allocate it.

### 6.3 String intern pool

Strings are special in Java. String literals may be interned, and the JVM maintains a pool for some string values.

### 6.4 Finalization and cleanup

Java has `finalize()` but it is not recommended for modern code. Prefer `try-with-resources` and explicit cleanup.

---

## 7. Multithreading and concurrency in the JVM

Java threads are managed by the JVM and the operating system.

### 7.1 Thread model

Each Java thread has:

- A stack
- A program counter
- Thread-local state

### 7.2 Thread scheduling

The OS scheduler decides when threads execute. The JVM does not fully control preemption.

### 7.3 Memory visibility

The Java Memory Model defines how changes made by one thread are visible to others.

Key concepts:

- Happens-before relationship
- Volatile variables
- Synchronization

### 7.4 Locking and monitors

The JVM uses monitors for synchronization. `synchronized` blocks and methods use intrinsic locking.

---

## 8. Class initialization and static behavior

A class is initialized when it is first actively used.

This can happen when:

- A static field is accessed
- A static method is invoked
- An instance is created
- A class is referenced by reflection

### 8.1 Static initialization order

Static initializers run once per class. They execute in a well-defined order.

### 8.2 Initialization hazards

Incorrect initialization logic can lead to:

- Race conditions
- Partial initialization
- Deadlocks

---

## 9. Performance tuning mindset

Understanding the runtime helps in tuning.

### 9.1 Measure before changing

Use:

- JFR
- VisualVM
- JConsole
- GC logs
- Thread dumps

### 9.2 Common bottlenecks

- Excessive object allocation
- Unbounded caches
- Lock contention
- Long GC pauses
- High CPU in certain methods

### 9.3 Practical advice

- Use the right GC for the workload
- Avoid premature optimization
- Profile before tuning
- Favor immutable and lightweight objects

---

## 10. Example: runtime behavior of a simple program

```java
public class Example {
    public static void main(String[] args) {
        String name = "Java";
        System.out.println(name.length());
    }
}
```

What happens at runtime:

1. `Example.class` is loaded
2. The main thread starts
3. Stack frame is created for `main`
4. The string literal is resolved
5. Bytecode is executed
6. The method is compiled if hot enough
7. The program exits cleanly

---

## 11. Key takeaways

The Java execution engine is a sophisticated system that turns bytecode into efficient native execution while managing memory, threads, and optimization automatically.

The most important concepts are:

- Class loading and initialization
- Heap, stack, metaspace, and GC
- JIT compilation and hot methods
- Threading and memory visibility
- Runtime profiling and tuning

A strong understanding of these mechanisms makes you a better Java engineer because you can reason about performance, memory, and reliability with far more precision.
