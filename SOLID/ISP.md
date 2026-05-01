---
# 🧠 Interface Segregation Principle (ISP) — 4 Levels
---

## 👶 1. Beginner Level (Simple Understanding)

👉 Rule:

Don’t force a class to use methods it doesn’t need

---

### 🎯 Example

Bad:

Machine → print(), scan(), fax()

👉 But a simple printer only needs print

---

👉 Simple line:

**Small interfaces are better than big ones**

---

## 👨‍💻 2. Mid-Level (Developer Understanding)

---

### 📌 Definition

Interface Segregation Principle says:

> A class should not be forced to implement interfaces it does not use

---

### ❌ Wrong Example (Fat Interface)

interface Machine {
public function print();
public function scan();
public function fax();
}

class Printer implements Machine {
public function print() {}

```
public function scan() {
    throw new Exception("Not supported"); // ❌
}

public function fax() {
    throw new Exception("Not supported"); // ❌
}
```

}

---

👉 Problem:

- Printer forced to implement unwanted methods
- Bad design

---

### ✅ Correct Design

interface PrinterInterface {
public function print();
}

interface ScannerInterface {
public function scan();
}

interface FaxInterface {
public function fax();
}

class SimplePrinter implements PrinterInterface {
public function print() {}
}

class MultiFunctionPrinter implements PrinterInterface, ScannerInterface, FaxInterface {
public function print() {}
public function scan() {}
public function fax() {}
}

---

👉 Now each class only implements what it needs

---

## 🧠 3. Expert Level (Deep Understanding)

---

### 🔥 Core Idea

👉 Break large interfaces into smaller, focused ones

---

### 🔥 Benefits

- Loose coupling
- Better flexibility
- Easy maintenance
- Clean architecture

---

### 🔥 Relation with other principles

- Supports SRP (single responsibility)
- Supports DIP (use abstraction)

---

### 🔥 Real-world examples

- Payment systems (Pay, Refund, Cancel अलग interfaces)
- User roles (Read, Write, Admin अलग permissions)
- APIs (separate endpoints/interfaces)

---

## 🧠 4. Critical Thinking (Interview Level)

---

### ❓ When NOT to apply ISP?

- very small systems
- simple interfaces
- no variation needed

---

### ❓ Drawbacks

- too many interfaces
- complexity increase
- harder navigation

---

### ❓ ISP vs SRP

| ISP                 | SRP                    |
| ------------------- | ---------------------- |
| Focus on interfaces | Focus on classes       |
| Split interfaces    | Split responsibilities |

---

### ❓ How to identify violation?

- class implements unused methods
- methods throw “not supported”
- interface too large

---

## ⚖️ Summary Table

| Level    | Understanding              |
| -------- | -------------------------- |
| Beginner | Don’t force methods        |
| Mid      | Split interfaces           |
| Expert   | Small focused abstractions |
| Critical | Balance complexity         |

---

## 🎯 Interview Answer

Interface Segregation Principle states that no client should be forced to depend on methods it does not use. It encourages creating smaller and more specific interfaces instead of large, general ones.

---

## 🧠 One-line

👉
**Many small interfaces are better than one large interface**
