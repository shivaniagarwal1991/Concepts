# Decorator Pattern (4 Levels Explanation)

---

## 1. Beginner Level (Simple Understanding)

👉 Idea:

Add extra behavior to an object **without changing its code**

---

### Example

Coffee:

- Basic Coffee
- Coffee + Milk
- Coffee + Sugar
- Coffee + Milk + Sugar

👉 Same object, but features बढ़ते जा रहे हैं

---

👉 Simple line:

Decorator = add features dynamically

---

## 2. Mid-Level (Developer Understanding)

---

### Definition

Decorator Pattern allows behavior to be added to an object dynamically without modifying its structure

---

### 🔧 PHP Example

---

#### Step 1: Interface

interface Coffee {
    public function cost();
}

---

#### Step 2: Base class

class BasicCoffee implements Coffee {
    public function cost() {
        return 10;
    }
}

---

#### Step 3: Decorator base

abstract class CoffeeDecorator implements Coffee {
    protected $coffee;

public function \_\_construct(Coffee $coffee) {
        $this->coffee = $coffee;
    }
}

---

#### Step 4: Add features

class MilkDecorator extends CoffeeDecorator {
    public function cost() {
        return $this->coffee->cost() + 5;
    }
}

class SugarDecorator extends CoffeeDecorator {
    public function cost() {
        return $this->coffee->cost() + 2;
    }
}

---

#### Step 5: Usage

$coffee = new BasicCoffee();
$coffee = new MilkDecorator($coffee);
$coffee = new SugarDecorator($coffee);

echo $coffee->cost(); // 17

---

### Understanding

👉 BasicCoffee = base
👉 Milk/Sugar = decorators

👉 Features add होते जा रहे हैं

---

## 3. Expert Level (Deep Design Thinking)

---

### Without Decorator

class Coffee {
    function cost($milk, $sugar) {}
}

👉 Problems:

- too many conditions
- messy code
- hard to extend

---

### With Decorator

👉 हर feature अलग class

👉 Combine easily

---

### Benefits

- Open/Closed principle
- Flexible design
- Avoid subclass explosion
- Runtime behavior change

---

### Structure

Component → Concrete Component → Decorator → Concrete Decorators

---

### Relation with Composition

👉 Decorator uses composition:

object inside object

---

## 4. Critical Thinking (Interview Level)

---

### When NOT to use Decorator?

- simple system
- few features
- no dynamic behavior

---

### Drawbacks

- many small classes
- harder debugging
- complex structure

---

### Decorator vs Inheritance

| Decorator        | Inheritance  |
| ---------------- | ------------ |
| runtime behavior | compile-time |
| flexible         | rigid        |
| composition      | hierarchy    |

---

### Real-world use cases

- Logging
- Middleware
- UI components
- File streams

---

### Decorator vs Strategy

| Decorator         | Strategy         |
| ----------------- | ---------------- |
| adds behavior     | changes behavior |
| stacking possible | single selection |

---

## Summary Table

| Level    | Understanding     |
| -------- | ----------------- |
| Beginner | add features      |
| Mid      | dynamic behavior  |
| Expert   | flexible design   |
| Critical | when to use/avoid |

---

## Interview Answer

Decorator Pattern allows adding new functionality to an object dynamically without changing its existing code. It uses composition to wrap objects and extend behavior.

---

## One-line

👉 Decorator Pattern = add features without modifying original object

---
