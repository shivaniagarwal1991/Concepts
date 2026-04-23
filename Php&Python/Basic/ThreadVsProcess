# Process vs Thread

---

## Basic Concept

- **Process = independent program (with its own memory)**
- **Thread = a smaller execution unit inside a process (shared memory)**

---

## Process

- Separate memory
- Different instances

Process1 ≠ Process2

Static variables are **not shared**

---

## Thread

- Same memory is shared

Thread1 == Thread2 (same memory)

Static variables are shared → problems arise here

---

## Combining Both Cases

---

### Case A: Multi-process Server (Most common in PHP)

User1 → Process1 → static1  
User2 → Process2 → static2

- Safe
- No race condition
- Why? - Memory is separate

---

### Case B: Multi-threaded Server (Python, Java, Node, etc.)

Thread1 → static  
Thread2 → same static

- Race condition possible

---

## Visual Understanding

---

### PHP (Process-based)

[Process1]  
 static = 1

[Process2]  
 static = 1

Separate copies

---

### Python (Thread-based)

Shared memory:  
 static = 0

Thread1 → increment  
Thread2 → increment

Conflict ❗

---

## Important Point

Even in Python:

- Multi-process → ✔ Safe
- Multi-thread → ❌ Unsafe

---

## When is Static Thread-safe?

| Scenario      | Safe?     |
| ------------- | --------- |
| Multi-process | ✔ Safe    |
| Multi-thread  | ❌ Unsafe |

---

## Deep Comparison

| Feature       | Process         | Thread                      |
| ------------- | --------------- | --------------------------- |
| Memory        | Separate        | Shared                      |
| Speed         | Slow (heavy)    | Fast (lightweight)          |
| Isolation     | High            | Low                         |
| Communication | Hard (IPC)      | Easy                        |
| Crash impact  | Safe (isolated) | Risky (affects all threads) |

---

## Memory Structure

### Process Memory

Process  
 ├── Code  
 ├── Heap  
 ├── Stack

- Everything is private

---

### Thread Memory

Process  
 ├── Shared Heap  
 ├── Thread1 Stack  
 ├── Thread2 Stack

- Heap is shared
- Stack is separate

---

## 🏢 Real-world Analogy

### Process

Different houses

    - Each house is separate
    -  No direct sharing

---

### Thread

Same house, different rooms

    - Resources are shared

---

## Language Perspective

### PHP

- Mostly process-based
- Safer for shared state

---

### Python

- Supports both threading and multiprocessing
- Affected by GIL (Global Interpreter Lock)

---

## Key Problems in Threads

- Race condition
- Deadlock
- Synchronization issues

---

## Final Summary

- Process = isolated, safe, slower
- Thread = shared, fast, risky

```

```
