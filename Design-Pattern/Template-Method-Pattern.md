# 🧠 Template Method Pattern — Complete Deep Dive

---

## 👶 1. Beginner Level (Simple Idea)

👉 **Idea:**

Define a fixed process (steps) and allow subclasses to customize some steps

---

### 🎯 Example

Making tea:

1. Boil water
2. Add tea leaves
3. Pour into cup
4. Add sugar

👉 Steps fixed, but sugar/milk customization possible

---

👉 Simple line:

**Template Method = fixed steps + customizable parts**

---

## 👨‍💻 2. Mid-Level (Developer Understanding)

---

### 📌 Definition

Template Method Pattern:

> Defines the skeleton of an algorithm in a base class and allows subclasses to override specific steps without changing the structure

---

### 🔧 PHP Example

---

### Step 1: Abstract class (template)

abstract class Beverage {

```
// Template method (final to prevent override)
final public function prepare() {
    $this->boilWater();
    $this->brew();
    $this->pour();
    $this->addExtras();
}

protected function boilWater() {
    echo "Boiling water\n";
}

protected function pour() {
    echo "Pouring into cup\n";
}

// Steps to override
abstract protected function brew();
abstract protected function addExtras();
```

}

---

### Step 2: Concrete classes

class Tea extends Beverage {

```
protected function brew() {
    echo "Steeping tea\n";
}

protected function addExtras() {
    echo "Adding lemon\n";
}
```

}

class Coffee extends Beverage {

```
protected function brew() {
    echo "Brewing coffee\n";
}

protected function addExtras() {
    echo "Adding sugar and milk\n";
}
```

}

---

### Step 3: Usage

$tea = new Tea();
$tea->prepare();

---

### 🧠 Understanding

👉 Base class controls flow
👉 Subclasses customize steps

---

## 🧠 3. Expert Level (Deep Design Thinking)

---

### 🔥 Key Concepts

---

#### 1. Template Method

👉 Fixed algorithm

prepare()

---

#### 2. Hooks (optional methods)

👉 Subclass may override

---

#### 3. Abstract methods

👉 Must implement

---

### 🔥 Why use Template Method?

- Enforce workflow
- Avoid duplication
- Control algorithm structure

---

### 🔥 Pattern Structure

AbstractClass
→ template method
→ abstract steps
→ concrete steps

---

### 🔥 Real-world examples

- Payment processing flow
- File parsing (CSV, JSON)
- Authentication flow
- Data pipelines

---

### 🔥 Example (Authentication)

Login flow:

1. validate input
2. authenticate
3. create session

👉 Step 2 can vary (DB, OAuth, API)

---

## 🧠 4. Critical Thinking (Interview Level)

---

### ❓ Template vs Strategy

| Template Method       | Strategy          |
| --------------------- | ----------------- |
| inheritance-based     | composition-based |
| fixed algorithm       | dynamic algorithm |
| compile-time behavior | runtime behavior  |

| Pattern         | Idea                        |
| --------------- | --------------------------- |
| Template Method | Fix the algorithm structure |
| Decorator       | Add behavior dynamically    |

| Feature        | Template Method   | Decorator       |
| -------------- | ----------------- | --------------- |
| Approach       | Inheritance       | Composition     |
| Goal           | Control structure | Extend behavior |
| Flexibility    | Low               | High            |
| Runtime change | No                | Yes             |
| Coupling       | Tight             | Loose           |

| Pattern   | Focus              |
| --------- | ------------------ |
| Template  | structure          |
| Strategy  | behavior selection |
| Decorator | behavior extension |

---

👉 Insight:

- Template = structure fixed
- Strategy = behavior replaceable

---

### ❓ Drawbacks

- inheritance tight coupling
- hard to change structure
- deep hierarchy possible

---

### ❓ When NOT to use?

- highly dynamic behavior
- frequent algorithm changes
- prefer composition (Strategy)

---

### ❓ Common mistake

👉 putting too much logic in base class

---

### ❓ LSP relation

👉 Subclass must follow base contract

---

## ⚖️ Summary Table

| Level    | Understanding           |
| -------- | ----------------------- |
| Beginner | fixed steps             |
| Mid      | base class defines flow |
| Expert   | control + extensibility |
| Critical | when to use vs avoid    |

---

## 🎯 Interview Answer

Template Method Pattern defines the structure of an algorithm in a base class while allowing subclasses to override specific steps. It ensures consistency while enabling customization.

---

## 🧠 One-line

👉
**Template Method = fixed algorithm + customizable steps**

---

## 🧠 Final Insight

👉

- Use when flow is fixed
- Customize only parts
- Prefer Strategy if flexibility needed

---
