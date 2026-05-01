# 🧠 Liskov Substitution Principle (LSP) — Complete Explanation

---

## 👶 1. Beginner Level (Simple Understanding)

👉 Rule:

Child class should be used in place of parent class without breaking anything

---

### 🎯 Example

Bird → can fly

Problem:

Penguin → cannot fly ❌

👉 So making Penguin a Bird (with fly behavior) is a bad design

---

### 🧠 Simple Idea

Parent and Child should behave the same way

---

## 👨‍💻 2. Mid-Level (Developer Understanding)

---

### 📌 Definition

LSP states:

A subclass should be replaceable for its parent class without breaking functionality

---

### ❌ Wrong Example

class Bird {
public function fly() {
echo "Flying";
}
}

class Penguin extends Bird {
public function fly() {
throw new Exception("Cannot fly"); // ❌ violates LSP
}
}

---

### ❌ Problem

- Parent says fly is possible
- Child says fly is not possible

👉 This breaks the contract

---

### ✅ Correct Design

class Bird {}

class FlyingBird extends Bird {
public function fly() {}
}

class Penguin extends Bird {}

---

👉 Now design is safe
👉 No broken behavior

---

## 🧠 3. Expert Level (Deep Understanding)

---

### 🔥 LSP = Contract + Behavior

👉 Not just methods
👉 Behavior must also match

---

### 📌 Example (Bank Account)

class Account {
public function withdraw($amount) {}
}

---

### ❌ Wrong Design

class FixedDeposit extends Account {
public function withdraw($amount) {
throw new Exception("Not allowed"); // ❌ violates LSP
}
}

---

👉 Parent allows withdrawal
👉 Child blocks it

👉 Design is wrong

---

### Better Design

interface Withdrawable {
public function withdraw($amount);
}

---

👉 Correct abstraction solves the issue

---

### Important Rules

1. Child should not break parent behavior
2. Child should not be more strict than parent
3. Child can improve results but not reduce functionality
4. Exceptions should not change unexpectedly

---

## 4. Critical Thinking (Interview Level)

---

### Is inheritance always correct?

❌ No

👉 Not every inheritance follows LSP

---

### Real-world problems

- Payment systems
- User roles
- File handling

---

### Why LSP is important?

- Prevents runtime errors
- Improves reliability
- Ensures clean architecture

---

### How to identify violation?

- Child throws errors where parent works
- Behavior changes unexpectedly
- System breaks when child is used

---

### Solution

- Use better abstraction
- Use interfaces
- Avoid incorrect inheritance

---

## Summary Table

| Level    | Understanding         |
| -------- | --------------------- |
| Beginner | Replaceable           |
| Mid      | Same behavior         |
| Expert   | Contract + behavior   |
| Critical | Avoid bad inheritance |

---

## Interview Answer

LSP states that a subclass should be replaceable for its parent class without affecting the correctness of the program. It ensures proper behavior and reliable inheritance design.

---

## One-line

👉 Child should behave like Parent without breaking the system
