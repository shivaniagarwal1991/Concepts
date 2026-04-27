# Interface vs Abstract Class — PHP vs Python

### Explained for Everyone: 5-Year-Old → Mid Dev → Expert → Critics

---

## Table of Contents

1. [The Core Idea — What Are These Things?](#1-the-core-idea)
2. [For a 5-Year-Old 🧸](#2-for-a-5-year-old-)
3. [For a Mid-Level Developer 💻](#3-for-a-mid-level-developer-)
4. [For an Expert Developer 🧠](#4-for-an-expert-developer-)
5. [For the Critics 🔥](#5-for-the-critics-)
6. [PHP vs Python — Full Comparison Table](#6-php-vs-python--full-comparison-table)
7. [When to Use What — Decision Guide](#7-when-to-use-what--decision-guide)

---

## 1. The Core Idea

Before diving into levels, here is the one-sentence truth about each:

| Concept            | One-Line Truth                                                                      |
| ------------------ | ----------------------------------------------------------------------------------- |
| **Interface**      | A **contract** — says WHAT must be done, never HOW                                  |
| **Abstract Class** | A **half-built blueprint** — says WHAT must be done AND can show HOW for some parts |

Both exist to enforce structure on classes that extend or implement them.
The difference is **how much they do themselves**.

---

## 2. For a 5-Year-Old 🧸

### The Toy Box Story

Imagine you have a toy box with different animals: a Dog, a Cat, and a Duck.

You want ALL of them to be able to `makeSound()` and `move()`.

---

### Interface = A Pinky Promise 🤙

An **Interface** is like a **pinky promise** you make with your toys.

> "I PROMISE every animal in this box will make a sound and move. But I won't tell them HOW — they figure it out themselves."

The Dog goes **"Woof"** and **runs**.
The Cat goes **"Meow"** and **sneaks**.
The Duck goes **"Quack"** and **waddles**.

They ALL kept the promise — but in their own way!

```
Interface Animal {
    makeSound()   ← just the promise, no details
    move()        ← just the promise, no details
}
```

---

### Abstract Class = A Half-Built Robot 🤖

An **Abstract Class** is like a **half-built robot** from a factory.

The factory already put in the **legs** (all robots walk the same way).
But it left a **hole** where the mouth should go — because every robot speaks differently.

> "I already gave you legs. You MUST add your own mouth. That part is NOT optional."

```
Abstract Robot {
    walk()        ← already built! all robots use this
    speak()       ← EMPTY HOLE — you MUST fill this yourself
}
```

So a **Dog Robot** adds barking. A **Duck Robot** adds quacking.
But they ALL walk the same way — because the factory already built that part.

---

### Simple Summary for 5-Year-Old

|                        | Interface       | Abstract Class             |
| ---------------------- | --------------- | -------------------------- |
| What it is             | A pinky promise | A half-built robot         |
| Does any work?         | ❌ Never        | ✅ Some parts, yes         |
| Who fills in the rest? | You — always    | You — only the empty holes |

---

## 3. For a Mid-Level Developer 💻

### PHP — Interface

In PHP, an interface uses the `interface` keyword. Every method declared inside **must be implemented** by the class that uses it. No method bodies allowed inside the interface itself.

```php
<?php

interface PaymentGateway
{
    public function charge(float $amount): bool;
    public function refund(float $amount): bool;
    public function getTransactionId(): string;
}

// Stripe implements ALL methods — no choice
class StripeGateway implements PaymentGateway
{
    public function charge(float $amount): bool
    {
        // Call Stripe API
        echo "Charging $$amount via Stripe\n";
        return true;
    }

    public function refund(float $amount): bool
    {
        echo "Refunding $$amount via Stripe\n";
        return true;
    }

    public function getTransactionId(): string
    {
        return 'stripe_' . uniqid();
    }
}

// PayPal implements ALL methods too — different way
class PayPalGateway implements PaymentGateway
{
    public function charge(float $amount): bool
    {
        echo "Charging $$amount via PayPal\n";
        return true;
    }

    public function refund(float $amount): bool
    {
        echo "Refunding $$amount via PayPal\n";
        return true;
    }

    public function getTransactionId(): string
    {
        return 'paypal_' . uniqid();
    }
}

// The magic: your code doesn't care WHICH gateway — just that it's a PaymentGateway
function processOrder(PaymentGateway $gateway, float $amount): void
{
    $gateway->charge($amount);
    echo "Transaction: " . $gateway->getTransactionId() . "\n";
}

processOrder(new StripeGateway(), 99.99);
processOrder(new PayPalGateway(), 49.99);
```

**Key rules in PHP interfaces:**

- All methods are automatically `public`
- No method bodies (no `{}` with code)
- A class can implement **multiple** interfaces
- Cannot have properties (only constants)

---

### PHP — Abstract Class

Abstract class uses the `abstract` keyword. It can have **both** abstract methods (empty — must be implemented) and **concrete** methods (already implemented — shared by all children).

```php
<?php

abstract class Notification
{
    // CONCRETE method — already implemented, shared by all children
    public function send(string $message): void
    {
        $formatted = $this->formatMessage($message); // calls abstract method
        $this->deliver($formatted);                   // calls abstract method
        $this->logSent($message);                     // shared logging — no override needed
    }

    // CONCRETE method — shared behaviour
    private function logSent(string $message): void
    {
        echo "[LOG] Notification sent: $message\n";
    }

    // ABSTRACT methods — children MUST implement these
    abstract protected function formatMessage(string $message): string;
    abstract protected function deliver(string $formatted): void;
}

class EmailNotification extends Notification
{
    protected function formatMessage(string $message): string
    {
        return "<html><body>$message</body></html>";
    }

    protected function deliver(string $formatted): void
    {
        echo "Sending EMAIL: $formatted\n";
    }
}

class SMSNotification extends Notification
{
    protected function formatMessage(string $message): string
    {
        // SMS: keep it short, strip HTML
        return strtoupper(substr($message, 0, 160));
    }

    protected function deliver(string $formatted): void
    {
        echo "Sending SMS: $formatted\n";
    }
}

$email = new EmailNotification();
$email->send("Your order has shipped!");

$sms = new SMSNotification();
$sms->send("Your order has shipped!");
```

**Output:**

```
Sending EMAIL: <html><body>Your order has shipped!</body></html>
[LOG] Notification sent: Your order has shipped!
Sending SMS: YOUR ORDER HAS SHIPPED!
[LOG] Notification sent: Your order has shipped!
```

Notice: `logSent()` ran for both — written once, shared everywhere. That is the power of abstract classes.

---

### Python — Interface

Python has **no native `interface` keyword**. There are two common approaches:

**Approach 1: ABC (Abstract Base Class) — most common**

```python
from abc import ABC, abstractmethod

class PaymentGateway(ABC):

    @abstractmethod
    def charge(self, amount: float) -> bool:
        pass

    @abstractmethod
    def refund(self, amount: float) -> bool:
        pass

    @abstractmethod
    def get_transaction_id(self) -> str:
        pass


class StripeGateway(PaymentGateway):

    def charge(self, amount: float) -> bool:
        print(f"Charging ${amount} via Stripe")
        return True

    def refund(self, amount: float) -> bool:
        print(f"Refunding ${amount} via Stripe")
        return True

    def get_transaction_id(self) -> str:
        import uuid
        return f"stripe_{uuid.uuid4().hex[:8]}"


class PayPalGateway(PaymentGateway):

    def charge(self, amount: float) -> bool:
        print(f"Charging ${amount} via PayPal")
        return True

    def refund(self, amount: float) -> bool:
        print(f"Refunding ${amount} via PayPal")
        return True

    def get_transaction_id(self) -> str:
        import uuid
        return f"paypal_{uuid.uuid4().hex[:8]}"


def process_order(gateway: PaymentGateway, amount: float) -> None:
    gateway.charge(amount)
    print(f"Transaction: {gateway.get_transaction_id()}")


process_order(StripeGateway(), 99.99)
process_order(PayPalGateway(), 49.99)

# This will raise TypeError — cannot instantiate ABC with abstract methods
# gateway = PaymentGateway()  ← ERROR: Can't instantiate abstract class
```

**Approach 2: Protocol (Python 3.8+) — structural subtyping / duck typing**

```python
from typing import Protocol

class PaymentGateway(Protocol):
    def charge(self, amount: float) -> bool: ...
    def refund(self, amount: float) -> bool: ...
    def get_transaction_id(self) -> str: ...

# Notice: StripeGateway does NOT inherit from PaymentGateway
# It just needs to have the same methods — Python checks the shape
class StripeGateway:
    def charge(self, amount: float) -> bool:
        return True

    def refund(self, amount: float) -> bool:
        return True

    def get_transaction_id(self) -> str:
        return "stripe_abc123"

# This works! Type checkers (mypy) verify the structure at check time
def process_order(gateway: PaymentGateway, amount: float) -> None:
    gateway.charge(amount)
```

---

### Python — Abstract Class

```python
from abc import ABC, abstractmethod

class Notification(ABC):

    # CONCRETE method — implemented here, shared by all
    def send(self, message: str) -> None:
        formatted = self.format_message(message)   # calls abstract
        self.deliver(formatted)                     # calls abstract
        self._log_sent(message)                     # shared, no override

    # CONCRETE private method — shared logging
    def _log_sent(self, message: str) -> None:
        print(f"[LOG] Notification sent: {message}")

    # ABSTRACT methods — subclasses MUST implement
    @abstractmethod
    def format_message(self, message: str) -> str:
        pass

    @abstractmethod
    def deliver(self, formatted: str) -> None:
        pass


class EmailNotification(Notification):

    def format_message(self, message: str) -> str:
        return f"<html><body>{message}</body></html>"

    def deliver(self, formatted: str) -> None:
        print(f"Sending EMAIL: {formatted}")


class SMSNotification(Notification):

    def format_message(self, message: str) -> str:
        return message[:160].upper()

    def deliver(self, formatted: str) -> None:
        print(f"Sending SMS: {formatted}")


email = EmailNotification()
email.send("Your order has shipped!")

sms = SMSNotification()
sms.send("Your order has shipped!")
```

---

### Mid-Level Summary

| Feature                 | PHP Interface | PHP Abstract           | Python ABC (Interface-style)  | Python Abstract Class          |
| ----------------------- | ------------- | ---------------------- | ----------------------------- | ------------------------------ |
| Keyword                 | `interface`   | `abstract class`       | `ABC` + all `@abstractmethod` | `ABC` + some `@abstractmethod` |
| Multiple inheritance    | ✅ Yes        | ❌ No                  | ✅ Yes (multiple ABC)         | ✅ Yes (MRO)                   |
| Concrete methods        | ❌ No         | ✅ Yes                 | ❌ (if pure interface)        | ✅ Yes                         |
| Properties              | ❌ No         | ✅ Yes                 | ✅ Yes                        | ✅ Yes                         |
| Enforces implementation | ✅ Yes        | ✅ Yes (abstract only) | ✅ Yes                        | ✅ Yes (abstract only)         |

---

## 4. For an Expert Developer 🧠

### PHP — Advanced Interface Patterns

#### Interface Segregation Principle (ISP) — Don't Force What You Don't Need

```php
<?php

// ❌ BAD — Fat interface — forces every class to implement everything
interface Worker
{
    public function work(): void;
    public function eat(): void;    // robots don't eat!
    public function sleep(): void;  // robots don't sleep!
}

// ✅ GOOD — Segregated interfaces
interface Workable
{
    public function work(): void;
}

interface Eatable
{
    public function eat(): void;
}

interface Sleepable
{
    public function sleep(): void;
}

// Human implements all three — makes sense
class HumanWorker implements Workable, Eatable, Sleepable
{
    public function work(): void { echo "Human working\n"; }
    public function eat(): void { echo "Human eating\n"; }
    public function sleep(): void { echo "Human sleeping\n"; }
}

// Robot only implements what it actually does
class RobotWorker implements Workable
{
    public function work(): void { echo "Robot working 24/7\n"; }
    // No eat(), no sleep() — and that's perfectly fine
}
```

#### Interface with Constants + Type-safe Polymorphism

```php
<?php

interface Serializable
{
    const FORMAT_JSON = 'json';
    const FORMAT_XML  = 'xml';

    public function serialize(string $format = self::FORMAT_JSON): string;
    public function unserialize(string $data, string $format = self::FORMAT_JSON): static;
}

interface Cacheable
{
    public function getCacheKey(): string;
    public function getTtl(): int;
}

// A class CAN implement multiple interfaces — powerful combination
class UserProfile implements Serializable, Cacheable
{
    public function __construct(
        private readonly int $id,
        private readonly string $name
    ) {}

    public function serialize(string $format = self::FORMAT_JSON): string
    {
        return match($format) {
            self::FORMAT_JSON => json_encode(['id' => $this->id, 'name' => $this->name]),
            self::FORMAT_XML  => "<user><id>{$this->id}</id><name>{$this->name}</name></user>",
            default           => throw new \InvalidArgumentException("Unknown format: $format"),
        };
    }

    public function unserialize(string $data, string $format = self::FORMAT_JSON): static
    {
        $decoded = match($format) {
            self::FORMAT_JSON => json_decode($data, true),
            default           => throw new \InvalidArgumentException("Unsupported"),
        };
        return new static($decoded['id'], $decoded['name']);
    }

    public function getCacheKey(): string { return "user_profile_{$this->id}"; }
    public function getTtl(): int { return 3600; }
}
```

#### Template Method Pattern with Abstract Classes

```php
<?php

// Abstract class enforces the algorithm skeleton
// Subclasses fill in the steps — classic Template Method
abstract class ReportGenerator
{
    // Template method — defines the algorithm, not overridable
    final public function generate(array $data): string
    {
        $data     = $this->validateData($data);
        $filtered = $this->filterData($data);
        $sorted   = $this->sortData($filtered);
        $rendered = $this->render($sorted);
        return $this->addHeader($rendered) . $rendered . $this->addFooter();
    }

    // Concrete with default — can be overridden if needed
    protected function validateData(array $data): array
    {
        return array_filter($data, fn($row) => !empty($row));
    }

    protected function sortData(array $data): array
    {
        return $data; // default: no sort
    }

    // Abstract — must be implemented
    abstract protected function filterData(array $data): array;
    abstract protected function render(array $data): string;
    abstract protected function addHeader(string $content): string;
    abstract protected function addFooter(): string;
}

class HTMLReport extends ReportGenerator
{
    protected function filterData(array $data): array
    {
        return array_values($data); // no filter for HTML
    }

    protected function render(array $data): string
    {
        $rows = implode('', array_map(fn($row) => "<tr><td>{$row['name']}</td><td>{$row['value']}</td></tr>", $data));
        return "<table>$rows</table>";
    }

    protected function addHeader(string $content): string
    {
        return "<!DOCTYPE html><html><body>";
    }

    protected function addFooter(): string
    {
        return "</body></html>";
    }
}

class CSVReport extends ReportGenerator
{
    protected function filterData(array $data): array
    {
        return array_filter($data, fn($row) => ($row['value'] ?? 0) > 0);
    }

    protected function sortData(array $data): array
    {
        usort($data, fn($a, $b) => $b['value'] <=> $a['value']);
        return $data;
    }

    protected function render(array $data): string
    {
        return implode("\n", array_map(fn($row) => "{$row['name']},{$row['value']}", $data));
    }

    protected function addHeader(string $content): string { return "name,value\n"; }
    protected function addFooter(): string { return "\n"; }
}
```

---

### Python — Advanced Patterns

#### ABC with `__subclasshook__` — Virtual Subclasses

```python
from abc import ABC, abstractmethod

class Drawable(ABC):

    @abstractmethod
    def draw(self) -> None:
        pass

    # Virtual subclass check — any class with a `draw` method qualifies
    @classmethod
    def __subclasshook__(cls, subclass):
        if cls is Drawable:
            return hasattr(subclass, 'draw') and callable(subclass.draw)
        return NotImplemented


class Circle:
    def draw(self) -> None:
        print("Drawing circle")

# Circle never inherited from Drawable — but Python treats it as one!
print(isinstance(Circle(), Drawable))  # True — duck typing + ABC combined
print(issubclass(Circle, Drawable))    # True
```

#### Mixin Pattern — Python's Answer to Multiple Interface-like Inheritance

```python
from abc import ABC, abstractmethod


# Core interface-style ABC
class Persistable(ABC):
    @abstractmethod
    def to_dict(self) -> dict: ...

    @abstractmethod
    def from_dict(self, data: dict) -> "Persistable": ...


# Mixin — concrete reusable behaviour, no abstract methods
class JsonMixin:
    def to_json(self) -> str:
        import json
        return json.dumps(self.to_dict())  # type: ignore

    def from_json(self, json_str: str) -> "JsonMixin":
        import json
        return self.from_dict(json.loads(json_str))  # type: ignore


class TimestampMixin:
    def __init__(self, *args, **kwargs):
        super().__init__(*args, **kwargs)
        from datetime import datetime
        self.created_at = datetime.utcnow().isoformat()


# Compose everything together via Python's MRO
class UserProfile(TimestampMixin, JsonMixin, Persistable):

    def __init__(self, user_id: int, name: str):
        super().__init__()
        self.user_id = user_id
        self.name = name

    def to_dict(self) -> dict:
        return {"id": self.user_id, "name": self.name, "created_at": self.created_at}

    def from_dict(self, data: dict) -> "UserProfile":
        return UserProfile(data["id"], data["name"])


user = UserProfile(1, "Shivani")
print(user.to_json())
# {"id": 1, "name": "Shivani", "created_at": "2026-04-25T..."}
```

#### Protocol — Structural Typing (No Inheritance Needed)

```python
from typing import Protocol, runtime_checkable

@runtime_checkable
class Drawable(Protocol):
    def draw(self) -> None: ...
    def get_area(self) -> float: ...


# No inheritance — just matching shape
class Circle:
    def __init__(self, radius: float):
        self.radius = radius

    def draw(self) -> None:
        print(f"Drawing circle r={self.radius}")

    def get_area(self) -> float:
        import math
        return math.pi * self.radius ** 2


class Square:
    def __init__(self, side: float):
        self.side = side

    def draw(self) -> None:
        print(f"Drawing square s={self.side}")

    def get_area(self) -> float:
        return self.side ** 2


# Both satisfy the Protocol without inheriting it
shapes: list[Drawable] = [Circle(5.0), Square(4.0)]
for shape in shapes:
    shape.draw()
    print(f"Area: {shape.get_area():.2f}")

# Runtime check works because of @runtime_checkable
print(isinstance(Circle(1), Drawable))  # True
```

#### Abstract Class with `__init_subclass__` — Auto-Registry Pattern

```python
from abc import ABC, abstractmethod


class Plugin(ABC):
    _registry: dict[str, type] = {}

    def __init_subclass__(cls, plugin_name: str = "", **kwargs):
        super().__init_subclass__(**kwargs)
        if plugin_name:
            Plugin._registry[plugin_name] = cls

    @abstractmethod
    def execute(self, data: dict) -> dict:
        pass

    @classmethod
    def get(cls, name: str) -> "Plugin":
        if name not in cls._registry:
            raise KeyError(f"Plugin '{name}' not registered")
        return cls._registry[name]()


class UppercasePlugin(Plugin, plugin_name="uppercase"):
    def execute(self, data: dict) -> dict:
        return {k: v.upper() if isinstance(v, str) else v for k, v in data.items()}


class TrimPlugin(Plugin, plugin_name="trim"):
    def execute(self, data: dict) -> dict:
        return {k: v.strip() if isinstance(v, str) else v for k, v in data.items()}


# Auto-discovered from registry — no manual registration
plugin = Plugin.get("uppercase")
print(plugin.execute({"name": "shivani", "city": "berlin"}))
# {'name': 'SHIVANI', 'city': 'BERLIN'}
```

---

### PHP vs Python — Expert Comparison

| Feature                     | PHP                        | Python                                                         |
| --------------------------- | -------------------------- | -------------------------------------------------------------- |
| Pure Interface keyword      | `interface` ✅ native      | No native — use `ABC` with all `@abstractmethod` or `Protocol` |
| Multiple interface impl     | `implements A, B, C`       | Multiple inheritance / multiple ABC                            |
| Abstract enforcement        | Compile-time (fatal error) | Runtime (TypeError on instantiation)                           |
| Structural typing           | Not natively supported     | `Protocol` — first-class structural subtyping                  |
| Virtual subclasses          | Not supported              | `ABC.register()` + `__subclasshook__`                          |
| Mixin pattern               | Traits (different concept) | Mixins via MRO                                                 |
| Interface constants         | ✅ Yes                     | Not standard (use `ClassVar`)                                  |
| `final` on template methods | `final` keyword            | Name-mangling convention (`__method`)                          |

---

## 5. For the Critics 🔥

### Criticism 1 — PHP Interfaces Are Syntactic Over-Engineering

**The Argument:**

> "PHP interfaces give you a false sense of safety. You're just making the compiler shout at you when you forget a method. A well-named class and good documentation does the same job without the ceremony."

**Counter-argument:**
This is true for small codebases. But at team scale — when 5 developers are building 3 different payment gateways — an interface is a machine-enforced contract. Documentation rots. Interfaces don't. If you remove a method from the interface, every implementing class immediately breaks. That's the point.

The real criticism should be at **fat interfaces** — the violation of ISP is rampant in PHP codebases where one god-interface does everything. The tool isn't broken. The usage is.

---

### Criticism 2 — Python's ABC Is a Hack, Not a Feature

**The Argument:**

> "Python's `ABC` is bolted on. It wasn't part of the original design. `Protocol` is better but not enforced at runtime by default. You're basically duct-taping a static type system onto a dynamic language."

**Valid point:**
`ABC` was introduced in Python 2.6 as a workaround for the fact that Python never had interfaces. It works — but it feels ceremonial in a language built on duck typing.

`Protocol` (PEP 544, Python 3.8+) is the more Pythonic answer because it matches how Python actually works — structurally, not nominally. But it requires mypy or pyright to be meaningful at type-check time.

**The real tension:**
Python is a dynamically typed language that has been gradually retrofitted with static typing. `ABC` and `Protocol` sit at that uncomfortable intersection. Neither is wrong — they serve different purposes and different eras of Python usage.

---

### Criticism 3 — Abstract Classes Are the Wrong Default

**The Argument:**

> "Abstract classes create tight coupling between parent and children. Every time you change the abstract class, all children are affected. You're trading short-term convenience for long-term rigidity."

**The evidence:**
This is the **fragile base class problem** — a well-documented OOP pitfall. When a concrete method in the abstract class changes, every subclass either inherits the change silently (dangerous) or breaks (good, but painful at scale).

**The real lesson:**
Favour composition over inheritance. Abstract classes are powerful for **Template Method** pattern and genuinely shared behaviour. But using them simply to share a few utility methods — when a plain mixin or a service class would do — is the actual mistake.

---

### Criticism 4 — The PHP vs Python Interface Debate Is Moot in 2026

**The Argument:**

> "Interfaces solve an inheritance-model problem. Modern systems use dependency injection, duck typing, and service containers. The question of 'interface vs abstract' is an OOP relic."

**Partially valid:**
In microservices and functional-leaning Python code, you rarely see `ABC` subclasses. You pass callables, use dataclasses, lean on Protocols for type-checking at CI time, and move on.

In PHP/Symfony — the framework is built on interfaces. `Symfony\Component\HttpKernel\HttpKernelInterface`, `Psr\Container\ContainerInterface`, `Psr\Log\LoggerInterface` — the entire PSR ecosystem is interface-driven. Ignoring interfaces in PHP means fighting the framework.

**Verdict:**
The choice of tool is language-ecosystem-specific. Criticising PHP interfaces from a Python mindset (or vice versa) is category error.

---

### Criticism 5 — Abstract Classes Lie About Their Intent

**The Argument:**

> "An abstract class tries to be two things: an interface (I define the contract) and a base class (I share code). This dual personality causes confusion. You can never tell if a new method added to the abstract class is meant to be part of the contract or just a convenience."

**This is the strongest criticism.**
When a senior dev adds a `protected function logError()` to an abstract class — is that part of the contract? Is it overridable? Should subclasses use it? The abstract class has no way to communicate intent clearly.

**PHP's partial fix:**
Use `final` on concrete methods you don't want overridden. Use `abstract` on methods that must be implemented. Document the rest.

**Python's partial fix:**
Name-mangling (`__method`) makes a method private. `@property` with no setter makes it read-only. But there's no `final` equivalent for methods in standard Python (without third-party decorators).

**The honest conclusion:**
Interfaces have a single, clear job. Abstract classes have two jobs. Two jobs means more surface area for misuse — but also more power when used with discipline.

---

## 6. PHP vs Python — Full Comparison Table

| Feature               | PHP Interface         | PHP Abstract Class          | Python ABC (pure)        | Python Abstract Class      | Python Protocol           |
| --------------------- | --------------------- | --------------------------- | ------------------------ | -------------------------- | ------------------------- |
| Native keyword        | `interface`           | `abstract class`            | `ABC` (stdlib)           | `ABC` (stdlib)             | `Protocol` (typing)       |
| Method bodies         | ❌ Never              | ✅ Yes (concrete)           | ❌ Never                 | ✅ Yes (concrete)          | ❌ Never (stubs only)     |
| Properties            | ❌ No                 | ✅ Yes                      | ✅ Yes                   | ✅ Yes                     | ✅ Yes (structural)       |
| Constants             | ✅ Yes                | ✅ Yes                      | Use `ClassVar`           | Use `ClassVar`             | Use `ClassVar`            |
| Multiple impl/inherit | ✅ Yes                | ❌ Single only              | ✅ Yes                   | ✅ MRO                     | ✅ Structural             |
| Inheritance required  | ✅ Yes (`implements`) | ✅ Yes (`extends`)          | ✅ Yes                   | ✅ Yes                     | ❌ No — structural        |
| Enforcement time      | Fatal error (load)    | Fatal error (load)          | RuntimeError             | RuntimeError               | mypy / pyright            |
| Instantiatable        | ❌ Never              | ❌ Never                    | ❌ Never                 | ❌ Never                   | ✅ (it's just a hint)     |
| Virtual subclasses    | ❌ No                 | ❌ No                       | ✅ `register()`          | ✅ `register()`            | ✅ structural match       |
| Best for              | Pure contracts        | Shared behaviour + contract | Pure contracts in Python | Shared behaviour in Python | Duck typing + type safety |

---

## 7. When to Use What — Decision Guide

```
You want to define a CONTRACT that others must follow
        │
        ├── Multiple classes will implement it differently?
        │           └── YES → Use Interface (PHP) / Protocol (Python modern)
        │
        ├── You want runtime enforcement (not just mypy)?
        │           └── YES → Use ABC with all @abstractmethod (Python)
        │
        └── Some shared code/behaviour exists too?
                    └── YES → Use Abstract Class
                              │
                              ├── Shared logic is SMALL (1-2 methods)?
                              │       └── Consider Mixin instead
                              │
                              └── Shared logic is the CORE of the algorithm?
                                      └── Abstract Class is correct — use Template Method pattern
```

### Quick Rules

**Use Interface / Pure ABC when:**

- You are defining a public API boundary (PSR standards, SDK contracts)
- Multiple completely different classes share zero behaviour, only method signatures
- You want to implement multiple contracts on one class

**Use Abstract Class when:**

- You have a real algorithm skeleton with some shared steps (Template Method)
- Child classes share non-trivial concrete behaviour
- You need shared state (properties) alongside the contract

**Use Protocol (Python only) when:**

- You want duck typing with static type safety
- You cannot or do not want to modify existing classes to inherit from your base
- You are working in a functional or data-science context where ABC feels heavy

---

## Final One-Liner Summary

| Audience         | Summary                                                                                                                                                                                                                           |
| ---------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 🧸 5-year-old    | Interface = pinky promise. Abstract = half-built robot with holes.                                                                                                                                                                |
| 💻 Mid-level dev | Interface = pure contract, no code. Abstract = contract + shared code.                                                                                                                                                            |
| 🧠 Expert dev    | Interface enforces LSP across multiple implementors. Abstract enables Template Method with controlled shared state. Python adds Protocol for structural subtyping without nominal inheritance.                                    |
| 🔥 Critics       | Both are OOP constructs with well-known failure modes — fragile base class, fat interfaces, nominal vs structural typing wars. Use with intent, not habit. Composition still beats inheritance in most real-world cases at scale. |

---

_Happy Coding! 🚀_
