# Strategy Pattern (4 Levels Explanation)

---

## 1. Beginner Level (Simple Understanding)

Idea:

Choose behavior at runtime

---

### Example

Payment system:

- Card
- UPI
- Cash

You can choose any method while running the program

---

Simple line:

Strategy = different ways to do the same thing

---

## 2. Mid-Level (Developer Understanding)

---

### Definition

Strategy Pattern defines a family of algorithms and makes them interchangeable at runtime

---

### PHP Example

---

#### Step 1: Interface

interface PaymentStrategy {
    public function pay($amount);
}

---

#### Step 2: Concrete strategies

class CardPayment implements PaymentStrategy {
    public function pay($amount) {
        echo "Paid $amount using Card";
    }
}

class UpiPayment implements PaymentStrategy {
    public function pay($amount) {
        echo "Paid $amount using UPI";
    }
}

---

#### Step 3: Context class

class Payment {
    private $strategy;

public function \_\_construct(PaymentStrategy $strategy) {
        $this->strategy = $strategy;
    }

public function makePayment($amount) {
        $this->strategy->pay($amount);
    }
}

---

#### Step 4: Usage

$payment = new Payment(new CardPayment());
$payment->makePayment(100);

---

### Understanding

Payment class does not care how payment happens
It just calls the strategy

---

## 3. Expert Level (Deep Design Thinking)

---

### Without Strategy

if ($type == "card") {}
elseif ($type == "upi") {}
elseif ($type == "cash") {}

---

👉 Problems:

- messy code
- hard to extend
- violates Open/Closed principle

---

### With Strategy

Add new method:

class CryptoPayment implements PaymentStrategy {}

👉 No change needed in existing code

---

### Benefits

- Open/Closed principle
- Loose coupling
- Easy to extend
- Easy to test

---

### Structure

Context → Strategy Interface → Concrete Strategies

---

### Relation with Dependency Injection

$payment = new Payment(new CardPayment());

👉 Strategy is injected

---

## 4. Critical Thinking (Interview Level)

---

### When NOT to use Strategy?

- only one behavior
- simple logic
- no need for flexibility

---

### Drawbacks

- too many classes
- increased complexity
- over-engineering

---

### Strategy vs if-else

- small logic → if-else
- scalable system → Strategy

---

### Real-world use cases

- Payment systems
- Sorting algorithms
- Authentication methods
- Compression systems

---

### Strategy vs Factory

| Strategy           | Factory           |
| ------------------ | ----------------- |
| Defines behavior   | Creates objects   |
| Focus on execution | Focus on creation |

---

## Summary Table

| Level    | Understanding                 |
| -------- | ----------------------------- |
| Beginner | Choose behavior               |
| Mid      | Interchangeable algorithms    |
| Expert   | Follows OOP design principles |
| Critical | Know when to use              |

---

## Interview Answer

Strategy Pattern allows defining multiple algorithms and selecting one at runtime. It removes conditional logic and makes the system flexible and extensible.

---

## One-line

👉 Strategy Pattern = interchangeable behavior at runtime

---
