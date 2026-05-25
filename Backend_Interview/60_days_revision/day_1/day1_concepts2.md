# 8. JDK, JRE and JVM

---

# Java Architecture Overview

Java follows the principle:

> **“Write Once, Run Anywhere (WORA)”**

Java code can run on any operating system because Java code is not converted directly into machine code.  
Instead, it is converted into **Bytecode**, which runs on the **JVM**.

---

# Flow of Java Program Execution

```text
.java file
   ↓
javac compiler
   ↓
.bytecode (.class)
   ↓
JVM
   ↓
Machine Code
   ↓
Program Executes
```

---

# Step-by-Step Execution of Java Program

---

## Step 1: Write Java Source Code

File Example:

```java
public class Main {
    public static void main(String[] args) {
        System.out.println("Hello Java");
    }
}
```

Saved as:

```text
Main.java
```

This is called:

- Source Code
- Human-readable code

---

## Step 2: Compilation using javac

Command:

```bash
javac Main.java
```

### What happens here?

The **Java Compiler (`javac`)** converts:

```text
.java → .class
```

Generated file:

```text
Main.class
```

This `.class` file contains:

# Bytecode

Bytecode is:

- Intermediate code
- Platform independent
- Understood by JVM

Example:

```text
cafebabe00000034...
```

(Not human readable)

---

# Why Bytecode?

Because machine code differs for:

- Windows
- Mac
- Linux

Java solves this by generating **common bytecode**.

Then JVM converts it to machine-specific code.

---

## Step 3: JVM Executes Bytecode

Command:

```bash
java Main
```

Now JVM starts execution.

JVM performs:

1. Class Loading
2. Bytecode Verification
3. Memory Allocation
4. Interpretation / JIT Compilation
5. Garbage Collection
6. Execution

---

# JDK, JRE and JVM Relationship

```text
JDK
 └── JRE
      └── JVM
```

---

# 1. JVM (Java Virtual Machine)

## Definition

JVM is a virtual machine that:

- Executes Java Bytecode
- Converts bytecode into machine code
- Makes Java platform independent

---

# Responsibilities of JVM

## 1. Loads Classes

Loads `.class` files into memory.

Done by:

- Class Loader Subsystem

---

## 2. Verifies Bytecode

Checks:

- Security
- Valid syntax
- Illegal memory access

Prevents dangerous code execution.

---

## 3. Executes Bytecode

Using:

- Interpreter
- JIT Compiler

---

## 4. Memory Management

Handles:

- Heap Memory
- Stack Memory
- Method Area

---

## 5. Garbage Collection

Automatically removes unused objects.

```java
Student s = new Student();
s = null;
```

Old object becomes eligible for GC.

---

# JVM Architecture

```text
JVM
 ├── Class Loader
 ├── Runtime Data Areas
 │     ├── Heap
 │     ├── Stack
 │     ├── Method Area
 │     ├── PC Register
 │     └── Native Method Stack
 │
 ├── Execution Engine
 │     ├── Interpreter
 │     ├── JIT Compiler
 │     └── Garbage Collector
 │
 └── JNI + Native Libraries
```

---

# JVM Memory Areas

---

## 1. Heap Memory

Stores:

- Objects
- Instance variables

Example:

```java
Student s = new Student();
```

Object goes into heap.

### Features

- Shared among threads
- Garbage collected

---

## 2. Stack Memory

Stores:

- Method calls
- Local variables
- References

Example:

```java
int x = 10;
```

Stored in stack.

Each thread has its own stack.

---

## 3. Method Area

Stores:

- Class metadata
- Static variables
- Method bytecode

Example:

```java
static int count;
```

Stored in method area.

---

## 4. PC Register

Stores current executing instruction address.

Each thread has separate PC register.

---

## 5. Native Method Stack

Used for native methods written in:

- C
- C++

Example:

```java
System.loadLibrary();
```

---

# Class Loader Subsystem

Loads `.class` files dynamically.

---

# Types of Class Loaders

## 1. Bootstrap ClassLoader

Loads core Java classes.

Example:

```text
java.lang.*
```

---

## 2. Extension ClassLoader

Loads extension libraries.

---

## 3. Application ClassLoader

Loads application classes from classpath.

---

# Delegation Model

```text
Application Loader
        ↓
Extension Loader
        ↓
Bootstrap Loader
```

Parent gets priority first.

---

# Execution Engine

Responsible for executing bytecode.

Contains:

- Interpreter
- JIT Compiler
- Garbage Collector

---

# Interpreter

Reads bytecode line by line and executes.

### Problem

Slow because every line is interpreted repeatedly.

---

# JIT Compiler (Just In Time Compiler)

JIT improves performance.

---

# What JIT Does

JIT identifies frequently used code called:

# Hotspots

Then converts them into native machine code.

---

# Without JIT

```text
Bytecode → Interpreter → Execution
```

Repeated every time.

---

# With JIT

```text
Bytecode
   ↓
JIT Compiler
   ↓
Native Machine Code
   ↓
Faster Execution
```

---

# Advantages of JIT

- Faster execution
- Optimized performance
- Reduces repeated interpretation

---

# Interview Question

## Is Java Compiled or Interpreted?

Answer:

> Java is both compiled and interpreted.

- `javac` compiles source code into bytecode
- JVM interprets/JIT compiles bytecode into machine code

---

# Garbage Collector (GC)

Automatically removes unused objects.

---

# Why GC Needed?

Without GC:

- Memory leaks
- Manual memory management

---

# GC Process

```text
Object Created
      ↓
Object Becomes Unreachable
      ↓
Garbage Collector Removes It
```

---

# Example

```java
Student s = new Student();
s = null;
```

Now object becomes eligible for GC.

---

# Can We Force GC?

```java
System.gc();
```

But JVM decides whether to run GC or not.

---

# Types of Garbage Collectors

- Serial GC
- Parallel GC
- G1 GC
- ZGC
- Shenandoah

---

# JVM Languages

JVM supports:

- Java
- Kotlin
- Scala
- Groovy
- Clojure

Because all compile to bytecode.

---

# 2. JRE (Java Runtime Environment)

## Definition

JRE provides environment to run Java applications.

Contains:

```text
JRE = JVM + Libraries + Runtime Files
```

---

# Components of JRE

## 1. JVM

Runs bytecode.

---

## 2. Core Libraries

Examples:

```text
java.lang
java.util
java.io
java.sql
```

---

## 3. Supporting Files

Runtime configurations and resources.

---

# Purpose of JRE

Used when:

- You only want to RUN Java programs
- No development required

---

# Does JRE Contain Compiler?

❌ No

JRE does NOT contain:

```text
javac
```

---

# 3. JDK (Java Development Kit)

## Definition

JDK is complete package for Java development.

Contains:

```text
JDK = JRE + Development Tools
```

---

# JDK Contains

## 1. JRE

Runtime environment.

---

## 2. Compiler (`javac`)

Compiles Java code.

---

## 3. Development Tools

Examples:

| Tool    | Purpose                 |
| ------- | ----------------------- |
| javac   | Compiler                |
| java    | Runs application        |
| javadoc | Generates documentation |
| jar     | Creates JAR files       |
| jdb     | Debugger                |
| javap   | Bytecode disassembler   |

---

# Difference Between JDK, JRE and JVM

| Feature                   | JVM | JRE | JDK |
| ------------------------- | --- | --- | --- |
| Executes bytecode         | ✅  | ✅  | ✅  |
| Contains JVM              | ❌  | ✅  | ✅  |
| Contains Compiler         | ❌  | ❌  | ✅  |
| Used for Development      | ❌  | ❌  | ✅  |
| Used for Running Programs | ✅  | ✅  | ✅  |

---

# Important Interview Point

## Java is Platform Independent BUT JVM is Platform Dependent

Why?

Because:

- Bytecode is same everywhere
- But JVM implementation differs for OS

Example:

- Windows JVM
- Linux JVM
- Mac JVM

---

# What Happens Internally When You Run Java Program?

```text
Main.java
   ↓
javac compiler
   ↓
Main.class (bytecode)
   ↓
Class Loader loads class
   ↓
Bytecode Verifier checks security
   ↓
JVM Memory allocated
   ↓
Interpreter executes bytecode
   ↓
JIT compiles hotspot code
   ↓
Machine code runs
```

---

# Java Compilation vs Execution

| Phase               | Tool Used         |
| ------------------- | ----------------- |
| Compilation         | javac             |
| Bytecode Generation | javac             |
| Execution           | JVM               |
| Optimization        | JIT               |
| Memory Cleanup      | Garbage Collector |

---

# Important JVM Interview Questions

## What is JVM?

JVM is a virtual machine that executes Java bytecode.

---

## What is Bytecode?

Intermediate platform-independent code generated by compiler.

---

## Why Java is Platform Independent?

Because bytecode runs on JVM available for every OS.

---

## What is JIT Compiler?

Converts frequently executed bytecode into native machine code.

---

## What is Heap Memory?

Stores objects and instance variables.

---

## Difference Between Stack and Heap

| Stack                     | Heap              |
| ------------------------- | ----------------- |
| Stores local variables    | Stores objects    |
| Thread specific           | Shared            |
| Faster                    | Larger            |
| Auto cleaned after method | Garbage collected |

---

## What is Garbage Collection?

Automatic memory cleanup process.

---

## Can Java Run Without JVM?

❌ No

JVM is mandatory for executing bytecode.

---

## Why is Java Slower Than C++?

Because Java uses:

- JVM
- Interpretation
- Garbage Collection

Extra abstraction layer adds overhead.

---

## What is ClassLoader?

Loads `.class` files into JVM memory.

---

## What is JVM Written In?

Mostly:

- C
- C++

---

## What is JNI?

JNI = Java Native Interface

Allows Java to call native code.

---

## What is HotSpot JVM?

Most popular JVM implementation by Oracle.

Uses advanced JIT optimizations.

---

# JDK vs OpenJDK

| Oracle JDK           | OpenJDK                    |
| -------------------- | -------------------------- |
| Oracle build         | Open-source implementation |
| Commercial support   | Community driven           |
| Mostly same features | Mostly same features       |

---

# JAR File

JAR = Java Archive

Used to package:

- `.class`
- resources
- metadata

Command:

```bash
jar cf app.jar Main.class
```

Run:

```bash
java -jar app.jar
```

---

# Java Execution Example End-to-End

## Source File

```java
public class Main {
    public static void main(String[] args) {
        int a = 10;
        int b = 20;
        System.out.println(a + b);
    }
}
```

---

## Compile

```bash
javac Main.java
```

Generates:

```text
Main.class
```

---

## Run

```bash
java Main
```

---

## Internal Execution

```text
Class Loader loads Main.class
↓
Bytecode verifier checks code
↓
JVM allocates memory
↓
Interpreter executes bytecode
↓
JIT optimizes repeated code
↓
Output produced
```

---

# One-Line Definitions

| Term     | One-Line Definition                        |
| -------- | ------------------------------------------ |
| JVM      | Executes Java bytecode                     |
| JRE      | Environment to run Java programs           |
| JDK      | Complete toolkit to develop Java programs  |
| JIT      | Converts hotspot bytecode into native code |
| Bytecode | Platform-independent intermediate code     |
| GC       | Removes unused objects automatically       |

---

# Quick Revision Diagram

```text
JDK
 ├── JRE
 │     ├── JVM
 │     │     ├── Class Loader
 │     │     ├── Memory Areas
 │     │     ├── Execution Engine
 │     │     └── Garbage Collector
 │     │
 │     └── Java Libraries
 │
 └── Development Tools
       ├── javac
       ├── java
       ├── jar
       ├── javadoc
       └── jdb
```

---

# 9. Interview Quick Index

| Question                             | Section                      |
| ------------------------------------ | ---------------------------- |
| What is JVM?                         | JVM Section                  |
| What is JRE?                         | JRE Section                  |
| What is JDK?                         | JDK Section                  |
| Explain JVM Architecture             | JVM Architecture             |
| What is Bytecode?                    | Bytecode Section             |
| What is JIT Compiler?                | JIT Compiler                 |
| Java compiled or interpreted?        | JIT Section                  |
| Explain Heap vs Stack                | Memory Areas                 |
| What is Garbage Collection?          | GC Section                   |
| Explain ClassLoader                  | ClassLoader Section          |
| Why Java is platform independent?    | Platform Independent Section |
| What happens when Java program runs? | Internal Execution           |
| What is HotSpot JVM?                 | HotSpot JVM                  |
| Difference between JDK JRE JVM?      | Comparison Table             |
| What is JNI?                         | JNI Section                  |
| What is JAR file?                    | JAR Section                  |
| Explain Java execution flow          | Execution Flow               |
| What are JVM memory areas?           | Memory Areas                 |
| Explain interpreter vs JIT           | Execution Engine             |
