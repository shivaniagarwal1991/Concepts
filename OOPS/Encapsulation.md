# Pillar 1: Encapsulation

### Bundling Data + Behaviour · Hiding Internal State · Controlling Access

#### 5-Year-Old 🧸 → Mid-Level 💻 → Expert 🧠 → Critics 🔥

---

## Table of Contents

1. [One-Line Truth](#1-one-line-truth)
2. [For a 5-Year-Old 🧸](#2-for-a-5-year-old-)
3. [For a Mid-Level Developer 💻](#3-for-a-mid-level-developer-)
4. [For an Expert Developer 🧠](#4-for-an-expert-developer-)
5. [For the Critics 🔥](#5-for-the-critics-)
6. [Quick Reference](#6-quick-reference)

---

## 1. One-Line Truth

> **Encapsulation** = Bundle data and the methods that operate on it into one unit, and **hide** the internal details from the outside world.

Two things happening together:

- **Bundling** — data + behaviour in one class
- **Information Hiding** — controlling what the outside world can see and touch

---

## 2. For a 5-Year-Old 🧸

### The Medicine Bottle Story 💊

Imagine a **medicine bottle**.

Inside the bottle there are pills — **you cannot just reach in and grab however many you want**.

The bottle has a **cap with a counter**. Every time you open it, it clicks once and records how many times it was opened. And it only lets you take **one pill at a time**.

```
Medicine Bottle (Class)
│
├── 💊 pills = 30          ← HIDDEN inside — you cannot touch directly
├── 🔢 openCount = 0       ← HIDDEN inside — you cannot touch directly
│
├── takePill()             ← The ONLY door to get a pill
│     - decreases pills by 1
│     - increases openCount by 1
│     - returns the pill
│
└── getPillsLeft()         ← The ONLY door to see how many remain
```

**You cannot do:**

```
bottle.pills = 0        ← ❌ NOT ALLOWED — hidden
bottle.openCount = 999  ← ❌ NOT ALLOWED — hidden
```

**You CAN do:**

```
bottle.takePill()       ← ✅ The safe, controlled way
bottle.getPillsLeft()   ← ✅ The safe, controlled way
```

**Why?** Because if anyone could just set `pills = 9999` — the bottle becomes dangerous.
Encapsulation protects the integrity of the data inside.

---

## 3. For a Mid-Level Developer 💻

### WITHOUT Encapsulation — The Problem ❌

```php
<?php

// WITHOUT encapsulation — everything is public, nothing is protected
class BankAccount
{
    public string $owner;
    public float  $balance;      // ← anyone can set this to anything
    public array  $transactions; // ← anyone can wipe this
}

$account         = new BankAccount();
$account->owner  = "Shivani";
$account->balance = 1000.0;

// NOTHING stops this:
$account->balance     = -99999.0;  // ❌ negative balance — bank is broken
$account->transactions = [];        // ❌ audit trail wiped
$account->balance     = PHP_INT_MAX; // ❌ infinite money glitch

// You need to validate everywhere that USES the class
function deposit(BankAccount $account, float $amount): void
{
    if ($amount <= 0) throw new \InvalidArgumentException("Must be positive");
    $account->balance += $amount; // still unsafe — race conditions, no logging
}

// But anyone can bypass your function:
$account->balance += 99999; // skips your deposit() entirely
```

**Problems without encapsulation:**

- Invalid states are possible — negative balance, empty audit log
- Validation must be duplicated everywhere the class is used
- Any code anywhere can corrupt the object's data
- Cannot change internal representation without breaking all callers

---

### WITH Encapsulation — The Solution ✅

```php
<?php

class BankAccount
{
    private float  $balance;
    private array  $transactions = [];
    private string $owner;

    public function __construct(string $owner, float $initialBalance = 0.0)
    {
        if ($initialBalance < 0) {
            throw new \InvalidArgumentException("Initial balance cannot be negative");
        }
        $this->owner   = $owner;
        $this->balance = $initialBalance;
    }

    // ---- Public interface — the ONLY doors in ----

    public function deposit(float $amount): void
    {
        $this->validatePositiveAmount($amount);
        $this->balance += $amount;
        $this->recordTransaction('deposit', $amount);
    }

    public function withdraw(float $amount): void
    {
        $this->validatePositiveAmount($amount);

        if ($amount > $this->balance) {
            throw new \RuntimeException("Insufficient funds");
        }

        $this->balance -= $amount;
        $this->recordTransaction('withdrawal', $amount);
    }

    public function transfer(BankAccount $target, float $amount): void
    {
        $this->withdraw($amount);   // uses our own validated method
        $target->deposit($amount);  // uses target's validated method
    }

    // Getters — read-only access, no direct mutation
    public function getBalance(): float         { return $this->balance; }
    public function getOwner(): string          { return $this->owner; }
    public function getTransactions(): array    { return $this->transactions; } // copy — not reference

    public function getStatement(): string
    {
        $lines = ["Account: {$this->owner}", "Balance: €{$this->balance}", "---"];
        foreach ($this->transactions as $tx) {
            $lines[] = "{$tx['type']}: €{$tx['amount']} at {$tx['at']}";
        }
        return implode("\n", $lines);
    }

    // ---- Private — internal machinery, nobody else's business ----

    private function validatePositiveAmount(float $amount): void
    {
        if ($amount <= 0) {
            throw new \InvalidArgumentException("Amount must be positive, got: $amount");
        }
    }

    private function recordTransaction(string $type, float $amount): void
    {
        $this->transactions[] = [
            'type'   => $type,
            'amount' => $amount,
            'at'     => (new \DateTimeImmutable())->format('Y-m-d H:i:s'),
        ];
    }
}

// Usage — clean, safe, impossible to corrupt
$account = new BankAccount("Shivani", 1000.0);
$account->deposit(500.0);
$account->withdraw(200.0);

echo $account->getBalance(); // 1300.0

// These are now IMPOSSIBLE:
// $account->balance = -99999;      ← Fatal error: Cannot access private property
// $account->transactions = [];      ← Fatal error: Cannot access private property

echo $account->getStatement();
```

---

### Visibility Levels

```php
<?php

class VisibilityDemo
{
    public    string $publicProp    = "anyone";       // accessible everywhere
    protected string $protectedProp = "me + children"; // class + subclasses
    private   string $privateProp   = "only me";      // this class ONLY

    public    function publicMethod(): void    { echo "public\n"; }
    protected function protectedMethod(): void { echo "protected\n"; }
    private   function privateMethod(): void   { echo "private\n"; }
}

class Child extends VisibilityDemo
{
    public function test(): void
    {
        echo $this->publicProp;      // ✅
        echo $this->protectedProp;   // ✅ — subclass access
        // echo $this->privateProp;  // ❌ Fatal error

        $this->publicMethod();       // ✅
        $this->protectedMethod();    // ✅
        // $this->privateMethod();   // ❌ Fatal error
    }
}

$obj = new VisibilityDemo();
echo $obj->publicProp;       // ✅
// echo $obj->protectedProp; // ❌ Fatal error
// echo $obj->privateProp;   // ❌ Fatal error
```

---

### Readonly Properties (PHP 8.1+)

```php
<?php

class User
{
    // Set once in constructor — immutable after that
    public function __construct(
        public readonly int    $id,
        public readonly string $email,
        private string         $name    // mutable but private
    ) {}

    public function changeName(string $name): void
    {
        $this->name = $name; // ✅ — internal change allowed
    }

    public function getName(): string { return $this->name; }
}

$user = new User(1, "shivani@gmail.com", "Shivani");

echo $user->id;     // ✅ — readable publicly
echo $user->email;  // ✅ — readable publicly

// $user->id    = 2;  // ❌ Cannot modify readonly property
// $user->email = "x"; // ❌ Cannot modify readonly property

$user->changeName("Shivani Agarwal"); // ✅ — controlled internal change
```

---

### Property Accessors — PHP 8.4 Property Hooks (Preview)

```php
<?php
// PHP 8.4 introduces property hooks — get/set directly on properties
class Temperature
{
    public float $celsius {
        get => $this->celsius;
        set {
            if ($value < -273.15) {
                throw new \ValueError("Below absolute zero!");
            }
            $this->celsius = $value;
        }
    }

    public float $fahrenheit {
        get  => $this->celsius * 9/5 + 32;  // computed — no storage
    }

    public function __construct(float $celsius)
    {
        $this->celsius = $celsius; // runs the setter hook
    }
}

$t = new Temperature(100.0);
echo $t->celsius;    // 100
echo $t->fahrenheit; // 212

$t->celsius = -300;  // ❌ ValueError: Below absolute zero!
```

---

## 4. For an Expert Developer 🧠

### Immutable Objects — Encapsulation's Ultimate Form

```php
<?php

// Immutable Value Object — once created, never changes
// Every "change" returns a NEW object

final class Money
{
    public function __construct(
        private readonly int    $amount,   // in cents — no float precision issues
        private readonly string $currency
    ) {
        if ($amount < 0) {
            throw new \InvalidArgumentException("Money amount cannot be negative");
        }
        if (!in_array($currency, ['EUR', 'USD', 'GBP'], true)) {
            throw new \InvalidArgumentException("Unsupported currency: $currency");
        }
    }

    // "Mutating" operations return NEW instances — original unchanged
    public function add(Money $other): self
    {
        $this->assertSameCurrency($other);
        return new self($this->amount + $other->amount, $this->currency);
    }

    public function subtract(Money $other): self
    {
        $this->assertSameCurrency($other);
        $result = $this->amount - $other->amount;
        if ($result < 0) {
            throw new \RuntimeException("Result would be negative");
        }
        return new self($result, $this->currency);
    }

    public function multiply(float $factor): self
    {
        return new self((int) round($this->amount * $factor), $this->currency);
    }

    public function isGreaterThan(Money $other): bool
    {
        $this->assertSameCurrency($other);
        return $this->amount > $other->amount;
    }

    public function equals(Money $other): bool
    {
        return $this->amount === $other->amount
            && $this->currency === $other->currency;
    }

    public function format(): string
    {
        return number_format($this->amount / 100, 2) . ' ' . $this->currency;
    }

    public function getAmount(): int    { return $this->amount; }
    public function getCurrency(): string { return $this->currency; }

    private function assertSameCurrency(Money $other): void
    {
        if ($this->currency !== $other->currency) {
            throw new \InvalidArgumentException(
                "Currency mismatch: {$this->currency} vs {$other->currency}"
            );
        }
    }
}

$price    = new Money(9999, 'EUR');   // €99.99
$tax      = $price->multiply(0.19);  // €18.998 → rounded to €19.00
$total    = $price->add($tax);

echo $price->format();  // 99.99 EUR  ← original unchanged!
echo $tax->format();    // 19.00 EUR
echo $total->format();  // 118.99 EUR

// Safe to share, cache, pass around — cannot be mutated by anyone
```

---

### Law of Demeter — "Don't Talk to Strangers"

```php
<?php

// ❌ VIOLATION — reaching through multiple objects
class OrderController
{
    public function processOrder(Order $order): void
    {
        // Chaining through: order → customer → address → city → zipCode
        $zip = $order->getCustomer()->getAddress()->getCity()->getZipCode();
        // This exposes the ENTIRE internal structure of Order, Customer, Address, City
        // Any change to any of those classes breaks this line
    }
}

// ✅ CORRECT — Order exposes only what the controller needs
class Order
{
    public function getShippingZipCode(): string
    {
        // Internal navigation hidden — only result exposed
        return $this->customer->getAddress()->getCity()->getZipCode();
    }
}

class OrderController
{
    public function processOrder(Order $order): void
    {
        $zip = $order->getShippingZipCode(); // ✅ one hop only
    }
}
```

**Law of Demeter:** A method should only call methods on:

- `$this` itself
- Its own properties
- Objects passed as parameters
- Objects it creates itself

NOT on objects returned by other calls.

---

### Encapsulation + Magic Accessors

```php
<?php

// Dynamic encapsulation — control ALL property access uniformly
class StrictEntity
{
    private array   $data      = [];
    private array   $dirty     = [];
    private array   $allowed   = ['name', 'email', 'age'];

    public function __get(string $name): mixed
    {
        if (!in_array($name, $this->allowed, true)) {
            throw new \InvalidArgumentException("Unknown property: $name");
        }
        return $this->data[$name] ?? null;
    }

    public function __set(string $name, mixed $value): void
    {
        if (!in_array($name, $this->allowed, true)) {
            throw new \InvalidArgumentException("Unknown property: $name");
        }
        if ($this->data[$name] ?? null !== $value) {
            $this->dirty[$name] = true;
        }
        $this->data[$name] = $value;
    }

    public function __isset(string $name): bool
    {
        return isset($this->data[$name]);
    }

    public function __unset(string $name): void
    {
        throw new \RuntimeException("Cannot unset entity properties");
    }

    public function isDirty(): bool          { return !empty($this->dirty); }
    public function getDirtyFields(): array  { return array_keys($this->dirty); }
    public function toArray(): array         { return $this->data; }
}

$entity        = new StrictEntity();
$entity->name  = "Shivani";    // ✅ goes through __set
$entity->email = "s@test.com"; // ✅
// $entity->secret = "x";      // ❌ InvalidArgumentException

echo $entity->name;             // ✅ goes through __get
echo $entity->isDirty() ? "modified\n" : "clean\n"; // modified
```

---

### Encapsulation in Context: ORM Internal State

```php
<?php

// Doctrine-style entity — encapsulates persistence state
class Article
{
    private ?int               $id          = null;
    private string             $title;
    private string             $content;
    private \DateTimeImmutable $createdAt;
    private \DateTimeImmutable $updatedAt;
    private string             $status      = 'draft';
    private array              $tags        = [];

    public function __construct(string $title, string $content)
    {
        $this->title     = $this->validateTitle($title);
        $this->content   = $content;
        $this->createdAt = new \DateTimeImmutable();
        $this->updatedAt = new \DateTimeImmutable();
    }

    // Domain behaviour — not just getters/setters
    public function publish(): void
    {
        if ($this->status === 'published') {
            throw new \DomainException("Article already published");
        }
        if (strlen($this->content) < 100) {
            throw new \DomainException("Article too short to publish");
        }
        $this->status    = 'published';
        $this->updatedAt = new \DateTimeImmutable();
    }

    public function addTag(string $tag): void
    {
        $tag = strtolower(trim($tag));
        if (!in_array($tag, $this->tags, true)) {
            $this->tags[] = $tag;
        }
    }

    public function updateContent(string $newContent): void
    {
        if ($this->status === 'published') {
            throw new \DomainException("Cannot edit a published article");
        }
        $this->content   = $newContent;
        $this->updatedAt = new \DateTimeImmutable();
    }

    // Read-only access
    public function getId(): ?int                    { return $this->id; }
    public function getTitle(): string               { return $this->title; }
    public function getStatus(): string              { return $this->status; }
    public function isPublished(): bool              { return $this->status === 'published'; }
    public function getTags(): array                 { return $this->tags; }

    private function validateTitle(string $title): string
    {
        $title = trim($title);
        if (strlen($title) < 3) throw new \InvalidArgumentException("Title too short");
        if (strlen($title) > 255) throw new \InvalidArgumentException("Title too long");
        return $title;
    }
}
```

---

## 5. For the Critics 🔥

### Criticism 1 — Getters and Setters Are NOT Encapsulation

**The Argument:**

> "Most PHP classes have private properties with `getX()` and `setX()` for every one. This is not encapsulation — it is encapsulation theatre. A public property and a getter/setter pair are functionally identical. The data is just as exposed."

**Strongly valid.**
`private $balance` with `getBalance()` and `setBalance()` is no better than `public $balance`. True encapsulation means NO setter for sensitive state, and mutation only through **domain methods** with business rules enforced inside.

The correct version: `deposit()`, `withdraw()`, `transfer()` — not `setBalance()`.
If you have a setter — ask: what rule does it enforce? If none — delete it.

---

### Criticism 2 — Encapsulation Creates Anemic Domain Models

**The Argument:**

> "In practice, developers create classes with private properties and public getters/setters, then put all the business logic in Service classes. The domain object becomes a data bag. You have encapsulation of the wrong thing — the data is hidden but the behaviour lives elsewhere."

**Correct — this is the Anemic Domain Model anti-pattern.**
Martin Fowler calls it one of the most common OOP mistakes. True encapsulation means the class holds BOTH data AND the behaviour that operates on it. `Article::publish()` lives on `Article` — not in `ArticleService::publish(Article $a)`. When your entity has only getters and setters and your service has all the logic — you have procedural code wearing OOP clothing.

---

### Criticism 3 — PHP Cannot Truly Hide Data

**The Argument:**

> "PHP's `private` can be bypassed with Reflection. No real information hiding exists — just a naming convention enforced by the engine. In testing, mocking frameworks routinely access private state. True encapsulation is a language-level guarantee PHP cannot provide."

**Factually true — philosophically weak.**
Reflection is an explicit, deliberate escape hatch — not a normal code path. `private` is not a security mechanism — it is a **design contract**. It tells the compiler and every developer reading the code: "do not depend on this." The fact that you CAN bypass it with Reflection is irrelevant to its value as an architectural signal. By this argument, no language truly encapsulates because machine code access always exists at some level.

---

What is Anemic Model(weak design)?

👉 Only data + getters/setters

👉 ❌ No business logic

Bad design

```php
class BankAccount {
    private $balance;

    public function setBalance($balance) {
        $this->balance = $balance;
    }
}

```

Good design

```php
class BankAccount {
    private $balance = 0;

    public function deposit($amount) {
        if ($amount <= 0) {
            throw new Exception("Invalid amount");
        }
        $this->balance += $amount;
    }

    public function withdraw($amount) {
        if ($amount > $this->balance) {
            throw new Exception("Insufficient balance");
        }
        $this->balance -= $amount;
    }

    public function getBalance() {
        return $this->balance;
    }
}
```

- Setter mindset => "Set data directly"
- Domain mindset => "Perform action with rules"

---

## 6. Quick Reference

```
Encapsulation Checklist:
  ✅ No public mutable properties
  ✅ No setters for fields that should only be set once
  ✅ State change through domain methods with validation
  ✅ Private for implementation details
  ✅ Protected only when subclasses genuinely need access
  ✅ Immutable Value Objects for concepts that don't change
  ✅ Return copies of collections, not references
  ✅ Law of Demeter respected — one hop only
  ❌ No "setBalance()" — use "deposit()" and "withdraw()"
  ❌ No public $balance — even "readonly" if it needs validation
```

| Visibility  | Accessible From    | Use When               |
| ----------- | ------------------ | ---------------------- |
| `public`    | Everywhere         | Intentional API        |
| `protected` | Class + subclasses | Template method hooks  |
| `private`   | This class only    | Implementation details |
| `readonly`  | Everywhere (read)  | Set-once, immutable    |

---

_Next: [Pillar 2 — Abstraction →](./02_abstraction.md)_
