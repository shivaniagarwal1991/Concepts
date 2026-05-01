# Composition vs Inheritance in PHP

### With and Without Each — Deep Dive

#### For Everyone: 5-Year-Old 🧸 → Mid-Level 💻 → Expert 🧠 → Critics 🔥

---

## Table of Contents

1. [The Core Idea — One Line Each](#1-the-core-idea)
2. [For a 5-Year-Old 🧸](#2-for-a-5-year-old-)
3. [For a Mid-Level Developer 💻](#3-for-a-mid-level-developer-)
   - 3.1 Inheritance — What, Why, How
   - 3.2 WITHOUT Inheritance (the problem it solves)
   - 3.3 WITH Inheritance (the solution)
   - 3.4 Composition — What, Why, How
   - 3.5 WITHOUT Composition (the problem it solves)
   - 3.6 WITH Composition (the solution)
   - 3.7 Side-by-Side Same Problem, Two Solutions
4. [For an Expert Developer 🧠](#4-for-an-expert-developer-)
   - 4.1 The Fragile Base Class Problem
   - 4.2 The Gorilla-Banana Problem
   - 4.3 Diamond Problem in PHP
   - 4.4 Liskov Substitution Principle (LSP)
   - 4.5 Composition Patterns — Strategy, Decorator, Mixin via Traits
   - 4.6 Dependency Injection as Composition
   - 4.7 When Inheritance IS Correct
   - 4.8 Deep Real-World Refactor — Inheritance → Composition
5. [For the Critics 🔥](#5-for-the-critics-)
6. [Full Comparison Table](#6-full-comparison-table)
7. [Decision Guide](#7-decision-guide)

---

## 1. The Core Idea

| Concept         | One-Line Truth                                                                      |
| --------------- | ----------------------------------------------------------------------------------- |
| **Inheritance** | A class **IS A** thing — `Dog IS AN Animal` — child gets everything parent has      |
| **Composition** | A class **HAS A** thing — `Car HAS AN Engine` — class owns objects that do the work |

The famous rule:

> **"Favour composition over inheritance"** — Gang of Four, Design Patterns (1994)

This does not mean "never use inheritance."
It means: **reach for composition first. Use inheritance only when the IS-A relationship is real and stable.**

---

## 2. For a 5-Year-Old 🧸

### The LEGO Story 🧱

#### Inheritance = Copying the Whole Box

Imagine you have a **LEGO Robot** kit.

Your friend wants to make a **LEGO Robot that also flies**.

With **Inheritance**, your friend takes the **entire robot kit** (ALL pieces — legs, arms, head, everything) and then **adds wings on top**.

```
Robot Kit (parent)
  - legs   ✅
  - arms   ✅
  - head   ✅
  - body   ✅

FlyingRobot (child — gets ALL of the above + more)
  - legs   ✅ (inherited — but flying robots don't walk!)
  - arms   ✅ (inherited)
  - head   ✅ (inherited)
  - body   ✅ (inherited)
  - wings  ✅ (new)
```

**Problem:** The FlyingRobot has **legs it doesn't need**.
You got the legs whether you wanted them or not.
You cannot say "I only want the head and the wings."

---

#### Composition = Picking Only the Pieces You Need

With **Composition**, your friend goes to the **LEGO parts bin** and picks ONLY what they need:

```
MyFlyingRobot
  - head   ✅ (picked from bin)
  - wings  ✅ (picked from bin)
  - engine ✅ (picked from bin)

  ← NO legs (didn't need them — didn't take them)
```

**Each piece does one job.** You combine them however you want.

If tomorrow you want a **Swimming Robot** — you pick `head` + `fins` + `propeller`.
You don't have to take the entire Robot kit.

---

### Simple Summary for 5-Year-Old

|              | Inheritance                     | Composition                         |
| ------------ | ------------------------------- | ----------------------------------- |
| How it works | Copy the whole parent           | Pick only the pieces you need       |
| Flexibility  | Low — get everything or nothing | High — mix and match                |
| Problem      | Stuck with stuff you don't need | Need to connect the pieces yourself |
| Real world   | Child IS a parent               | Car HAS an engine                   |

---

## 3. For a Mid-Level Developer 💻

### 3.1 Inheritance — What, Why, How

Inheritance lets one class **extend** another — getting all its properties and methods.

```
Animal (parent / base class)
  ↑
  ├── Dog   (child — inherits from Animal)
  ├── Cat   (child — inherits from Animal)
  └── Bird  (child — inherits from Animal)
```

The child class:

- Gets everything from the parent
- Can ADD new methods/properties
- Can OVERRIDE parent methods

---

### 3.2 WITHOUT Inheritance — The Problem

Imagine you need to build `Dog` and `Cat` classes. Without any reuse:

```php
<?php

// WITHOUT inheritance — pure copy-paste hell ❌

class Dog
{
    private string $name;
    private int $age;

    public function __construct(string $name, int $age)
    {
        $this->name = $name;
        $this->age  = $age;
    }

    // DUPLICATED in every animal class ↓
    public function getName(): string { return $this->name; }
    public function getAge(): int     { return $this->age; }
    public function eat(): string     { return "{$this->name} is eating"; }
    public function sleep(): string   { return "{$this->name} is sleeping"; }
    // DUPLICATED in every animal class ↑

    // Dog-specific
    public function makeSound(): string { return "{$this->name} says: Woof!"; }
    public function fetch(): string     { return "{$this->name} fetches the ball!"; }
}

class Cat
{
    private string $name;
    private int $age;

    public function __construct(string $name, int $age)
    {
        $this->name = $name;
        $this->age  = $age;
    }

    // EXACT SAME CODE AGAIN — copy-pasted ↓
    public function getName(): string { return $this->name; }
    public function getAge(): int     { return $this->age; }
    public function eat(): string     { return "{$this->name} is eating"; }
    public function sleep(): string   { return "{$this->name} is sleeping"; }
    // EXACT SAME CODE AGAIN ↑

    // Cat-specific
    public function makeSound(): string { return "{$this->name} says: Meow!"; }
    public function purr(): string      { return "{$this->name} is purring..."; }
}

class Bird
{
    private string $name;
    private int $age;

    public function __construct(string $name, int $age)
    {
        $this->name = $name;
        $this->age  = $age;
    }

    // SAME CODE. AGAIN. A THIRD TIME. ↓
    public function getName(): string { return $this->name; }
    public function getAge(): int     { return $this->age; }
    public function eat(): string     { return "{$this->name} is eating"; }
    public function sleep(): string   { return "{$this->name} is sleeping"; }
    // SAME CODE. AGAIN. ↑

    public function makeSound(): string { return "{$this->name} says: Tweet!"; }
    public function fly(): string       { return "{$this->name} is flying!"; }
}

// Problems:
// 1. eat() and sleep() copied in 3 places — bug in one = fix in 3 places
// 2. Cannot do: function feedAll(array $animals) — Dog, Cat, Bird are unrelated types
// 3. Adding a new property (e.g. $weight) means updating ALL classes
```

**Problems without inheritance:**

- Massive code duplication
- Cannot treat Dog, Cat, Bird as the same type
- One bug fix = N places to change (N = number of animal classes)

---

### 3.3 WITH Inheritance — The Solution

```php
<?php

// WITH inheritance — shared code lives once ✅

abstract class Animal
{
    public function __construct(
        protected string $name,
        protected int    $age
    ) {}

    // Shared methods — written ONCE, used by all children
    public function getName(): string { return $this->name; }
    public function getAge(): int     { return $this->age; }
    public function eat(): string     { return "{$this->name} is eating"; }
    public function sleep(): string   { return "{$this->name} is sleeping"; }

    // Children MUST implement this differently
    abstract public function makeSound(): string;
}

class Dog extends Animal
{
    public function makeSound(): string { return "{$this->name} says: Woof!"; }
    public function fetch(): string     { return "{$this->name} fetches the ball!"; }
}

class Cat extends Animal
{
    public function makeSound(): string { return "{$this->name} says: Meow!"; }
    public function purr(): string      { return "{$this->name} is purring..."; }
}

class Bird extends Animal
{
    public function makeSound(): string { return "{$this->name} says: Tweet!"; }
    public function fly(): string       { return "{$this->name} is flying!"; }
}

// Now they share a common type — polymorphism works
$animals = [
    new Dog("Rex", 3),
    new Cat("Whiskers", 5),
    new Bird("Tweety", 2),
];

// Feed ALL animals — same function, any animal type
foreach ($animals as $animal) {
    echo $animal->eat() . "\n";
    echo $animal->makeSound() . "\n";
}

// Output:
// Rex is eating
// Rex says: Woof!
// Whiskers is eating
// Whiskers says: Meow!
// Tweety is eating
// Tweety says: Tweet!
```

**Benefits with inheritance:**

- `eat()` and `sleep()` written once — change in one place
- All animals share a type (`Animal`) — polymorphism works
- Adding `$weight` = update `Animal` only — all children get it

---

### 3.4 Composition — What, Why, How

Composition means a class **contains instances of other classes** — it delegates work to them.

```
Car has-an Engine
Car has-a  GPS
Car has-a  MusicPlayer
```

Instead of inheriting `Engine`'s behavior — Car **owns an Engine** and **calls its methods**.

---

### 3.5 WITHOUT Composition — The Problem

Imagine you need a notification system: Email, SMS, Push. Using only inheritance:

```php
<?php

// WITHOUT composition — inheritance used wrong ❌

class Notification
{
    protected string $message;

    public function __construct(string $message)
    {
        $this->message = $message;
    }

    public function send(): void
    {
        // How? This class doesn't know HOW to send anything!
        // So we duplicate in children...
    }
}

class EmailNotification extends Notification
{
    private string $toEmail;

    public function __construct(string $message, string $toEmail)
    {
        parent::__construct($message);
        $this->toEmail = $toEmail;
    }

    public function send(): void
    {
        // Email-specific sending logic — baked in, cannot be reused
        echo "Sending EMAIL to {$this->toEmail}: {$this->message}\n";
        // imagine 50 lines of SMTP logic here
    }
}

class SmsNotification extends Notification
{
    private string $phoneNumber;

    public function __construct(string $message, string $phoneNumber)
    {
        parent::__construct($message);
        $this->phoneNumber = $phoneNumber;
    }

    public function send(): void
    {
        // SMS-specific sending logic — baked in, cannot be reused
        echo "Sending SMS to {$this->phoneNumber}: {$this->message}\n";
        // imagine 50 lines of Twilio logic here
    }
}

// NOW: What if you need Email + SMS together? (MultiChannel)
// You'd have to create ANOTHER class and copy both send() implementations
// Or do crazy multiple inheritance workarounds
// What if the email sending provider changes? Rewrite EmailNotification entirely.
// What if you want to test WITHOUT actually sending? You cannot swap the sending logic.
```

**Problems without composition:**

- Sending logic is **locked inside** each class — cannot be swapped
- Cannot test without actually sending emails/SMS
- Multi-channel notification = copy-paste or hacky inheritance
- Change email provider = rewrite entire class

---

### 3.6 WITH Composition — The Solution

```php
<?php

// WITH composition — each piece does ONE job, assembled as needed ✅

// Step 1: Define the contract (interface)
interface MessageSender
{
    public function send(string $message, string $recipient): void;
}

// Step 2: Concrete senders — each does ONE thing
class EmailSender implements MessageSender
{
    public function send(string $message, string $recipient): void
    {
        echo "📧 EMAIL to {$recipient}: {$message}\n";
        // SMTP logic here — swappable, testable
    }
}

class SmsSender implements MessageSender
{
    public function send(string $message, string $recipient): void
    {
        echo "📱 SMS to {$recipient}: {$message}\n";
        // Twilio logic here — swappable, testable
    }
}

class PushSender implements MessageSender
{
    public function send(string $message, string $recipient): void
    {
        echo "🔔 PUSH to {$recipient}: {$message}\n";
        // Firebase logic here — swappable, testable
    }
}

// Step 3: Notification class COMPOSES senders
class Notification
{
    /** @var MessageSender[] */
    private array $senders;

    // Constructor injection — senders passed IN, not baked in
    public function __construct(MessageSender ...$senders)
    {
        $this->senders = $senders;
    }

    public function send(string $message, string $recipient): void
    {
        foreach ($this->senders as $sender) {
            $sender->send($message, $recipient);
        }
    }
}

// Usage — compose any combination you want
$emailOnly = new Notification(
    new EmailSender()
);

$emailAndSms = new Notification(
    new EmailSender(),
    new SmsSender()
);

$allChannels = new Notification(
    new EmailSender(),
    new SmsSender(),
    new PushSender()
);

$emailOnly->send("Your order shipped!", "shivani@gmail.com");
// 📧 EMAIL to shivani@gmail.com: Your order shipped!

$allChannels->send("Payment received!", "shivani@gmail.com");
// 📧 EMAIL to shivani@gmail.com: Payment received!
// 📱 SMS to shivani@gmail.com: Payment received!
// 🔔 PUSH to shivani@gmail.com: Payment received!

// Testing — swap real sender with mock — zero code change in Notification
class MockSender implements MessageSender
{
    public array $sent = [];

    public function send(string $message, string $recipient): void
    {
        $this->sent[] = ['message' => $message, 'recipient' => $recipient];
    }
}

$mock = new MockSender();
$notification = new Notification($mock);
$notification->send("Test message", "test@test.com");
assert($mock->sent[0]['message'] === "Test message"); // ✅ testable!
```

**Benefits with composition:**

- `EmailSender` can be swapped with any `MessageSender` — zero Notification change
- Multi-channel = just pass multiple senders
- Testing = pass a `MockSender`
- Change email provider = rewrite `EmailSender` only
- `Notification` has **no idea** HOW sending works — it just delegates

---

### 3.7 Side-by-Side — Same Problem, Two Solutions

**Problem:** Build a user system with logging and database saving.

#### With Inheritance (Wrong approach here)

```php
<?php

class DatabaseConnection
{
    public function save(array $data): void
    {
        echo "Saving to DB: " . json_encode($data) . "\n";
    }
}

class Logger extends DatabaseConnection  // ❌ Logger IS-A DatabaseConnection?? No sense!
{
    public function log(string $message): void
    {
        echo "[LOG] $message\n";
    }
}

class UserRepository extends Logger  // ❌ UserRepository IS-A Logger? No!
{
    public function createUser(string $name, string $email): void
    {
        $this->log("Creating user: $name");    // from Logger
        $this->save(['name' => $name, 'email' => $email]);  // from DatabaseConnection
    }
}

$repo = new UserRepository();
$repo->createUser("Shivani", "shivani@gmail.com");

// Problems:
// 1. The IS-A relationships are nonsense — a UserRepository is NOT a Logger
// 2. UserRepository now exposes save() and log() publicly — breaks encapsulation
// 3. Cannot change the logger without touching the inheritance chain
// 4. Cannot use a different DB connection — it's baked in via inheritance
```

#### With Composition (Right approach)

```php
<?php

// Each class does ONE job
interface LoggerInterface
{
    public function log(string $message): void;
}

interface DatabaseInterface
{
    public function save(array $data): void;
}

class ConsoleLogger implements LoggerInterface
{
    public function log(string $message): void
    {
        echo "[LOG] " . date('H:i:s') . " — $message\n";
    }
}

class MySQLDatabase implements DatabaseInterface
{
    public function save(array $data): void
    {
        echo "Saving to MySQL: " . json_encode($data) . "\n";
    }
}

// UserRepository HAS-A logger and HAS-A database — not IS-A
class UserRepository
{
    public function __construct(
        private readonly LoggerInterface    $logger,
        private readonly DatabaseInterface  $db
    ) {}

    public function createUser(string $name, string $email): void
    {
        $this->logger->log("Creating user: $name");
        $this->db->save(['name' => $name, 'email' => $email]);
        $this->logger->log("User created: $name");
    }
}

// Compose with real dependencies
$repo = new UserRepository(
    new ConsoleLogger(),
    new MySQLDatabase()
);

$repo->createUser("Shivani", "shivani@gmail.com");

// Compose with different dependencies — zero change to UserRepository
class FileLogger implements LoggerInterface
{
    public function log(string $message): void
    {
        file_put_contents('/var/log/app.log', "[LOG] $message\n", FILE_APPEND);
    }
}

class RedisDatabase implements DatabaseInterface
{
    public function save(array $data): void
    {
        echo "Saving to Redis: " . json_encode($data) . "\n";
    }
}

$repoV2 = new UserRepository(
    new FileLogger(),     // ← different logger
    new RedisDatabase()   // ← different DB
);

// UserRepository did not change AT ALL — only what we passed in changed
```

---

## 4. For an Expert Developer 🧠

### 4.1 The Fragile Base Class Problem

The most dangerous consequence of deep inheritance.

```php
<?php

class Collection
{
    protected array $items = [];
    protected int   $count = 0;

    public function add(mixed $item): void
    {
        $this->items[] = $item;
        $this->count++;           // manually tracking count
    }

    public function addAll(array $items): void
    {
        foreach ($items as $item) {
            $this->add($item);    // calls add() — relies on it
        }
    }

    public function count(): int
    {
        return $this->count;
    }
}

class CountingCollection extends Collection
{
    private int $addCallCount = 0;

    public function add(mixed $item): void
    {
        $this->addCallCount++;
        parent::add($item);       // calls parent
    }

    public function getAddCallCount(): int
    {
        return $this->addCallCount;
    }
}

$c = new CountingCollection();
$c->addAll(['a', 'b', 'c']);   // addAll calls add() 3 times

echo $c->getAddCallCount();    // Expect: 3 — Got: 3 ✅ (for now)
echo $c->count();              // 3 ✅

// NOW: Base class author "optimizes" addAll() — internal change
class Collection  // v2 — author changes implementation
{
    protected array $items = [];
    protected int   $count = 0;

    public function add(mixed $item): void
    {
        $this->items[] = $item;
        $this->count++;
    }

    public function addAll(array $items): void
    {
        // "Optimization" — directly push instead of calling add()
        array_push($this->items, ...$items);
        $this->count += count($items);    // ← NO LONGER CALLS add()!
    }

    public function count(): int { return $this->count; }
}

// CountingCollection is now SILENTLY BROKEN
$c = new CountingCollection();
$c->addAll(['a', 'b', 'c']);

echo $c->getAddCallCount();  // Expected: 3 — Got: 0 ❌ SILENT BUG
echo $c->count();            // 3 ✅ (count is correct, but override didn't fire)

// The child class broke because the parent changed an internal implementation detail
// The parent author had no idea they were breaking a child class
// The child class author had no idea addAll() would stop calling add()
// This is the Fragile Base Class Problem
```

#### Solution with Composition

```php
<?php

class Collection
{
    private array $items = [];

    public function add(mixed $item): void
    {
        $this->items[] = $item;
    }

    public function addAll(array $items): void
    {
        foreach ($items as $item) {
            $this->add($item);
        }
    }

    public function count(): int { return count($this->items); }
    public function all(): array { return $this->items; }
}

// CountingCollection WRAPS Collection — not extends it
class CountingCollection
{
    private Collection $collection;
    private int        $addCallCount = 0;

    public function __construct()
    {
        $this->collection = new Collection();  // HAS-A, not IS-A
    }

    public function add(mixed $item): void
    {
        $this->addCallCount++;
        $this->collection->add($item);  // delegate
    }

    public function addAll(array $items): void
    {
        foreach ($items as $item) {
            $this->add($item);           // calls OUR add() — always
        }
    }

    public function count(): int          { return $this->collection->count(); }
    public function getAddCallCount(): int { return $this->addCallCount; }
}

$c = new CountingCollection();
$c->addAll(['a', 'b', 'c']);

echo $c->getAddCallCount();  // 3 ✅ — always correct, regardless of Collection internals
echo $c->count();            // 3 ✅

// Collection can change its internal addAll() implementation all it wants
// CountingCollection is immune — it controls its own addAll()
```

---

### 4.2 The Gorilla-Banana Problem

> "You wanted a banana. What you got was a gorilla holding the banana, and the entire jungle."
> — Joe Armstrong, creator of Erlang

```php
<?php

// The gorilla-banana problem in PHP
class Animal
{
    protected string $name;
    protected string $habitat;
    protected float  $bodyTemperature;
    protected bool   $isEndangered;
    protected array  $dietaryRestrictions;
    protected string $kingdom;
    protected string $phylum;
    protected string $class;
    protected string $order;
    protected string $family;
    protected string $genus;
    // ... 40 more properties

    public function eat(): string { /* complex digestion logic */ return ''; }
    public function sleep(): void { /* circadian rhythm logic */ }
    public function reproduce(): void { /* complex logic */ }
    public function migrate(): void { /* GPS-based movement logic */ }
    // ... 30 more methods
}

class Primate extends Animal
{
    protected bool  $hasOpposableThumbs;
    protected float $brainToBodyRatio;
    // ... more primate stuff
    public function useTools(): void { /* complex logic */ }
    // ... more primate methods
}

class Gorilla extends Primate
{
    protected float $silverbackDominanceScore;
    // ... gorilla stuff
    public function beatChest(): void { /* complex logic */ }
}

// All you wanted was:
class Banana extends Gorilla  // ❌ You wanted a Banana — you got the ENTIRE JUNGLE
{
    // Banana has: habitat, bodyTemperature, isEndangered, dietaryRestrictions,
    //             kingdom, phylum, class, order, family, genus, hasOpposableThumbs,
    //             brainToBodyRatio, silverbackDominanceScore, eat(), sleep(),
    //             reproduce(), migrate(), useTools(), beatChest()...
    //             and you just wanted yellow and sweet!

    private string $color  = 'yellow';
    private string $flavor = 'sweet';
}

// With composition:
class Banana
{
    public function __construct(
        private string $color  = 'yellow',
        private string $flavor = 'sweet'
    ) {}

    public function getColor(): string  { return $this->color; }
    public function getFlavor(): string { return $this->flavor; }
}

// Got exactly what you needed. No jungle.
```

---

### 4.3 Diamond Problem in PHP

PHP does not support multiple class inheritance — but it can simulate the diamond problem via interfaces and traits.

```php
<?php

// The Diamond Problem — why PHP disallows multiple inheritance
//
//        Animal
//       /      \
//    Flyable  Swimmable
//       \      /
//        Duck
//
// If both Flyable and Swimmable define move() — which one does Duck get?

interface Flyable
{
    public function move(): string;
}

interface Swimmable
{
    public function move(): string;
}

// ❌ This creates ambiguity
class Duck implements Flyable, Swimmable
{
    // PHP FORCES you to implement move() yourself — no ambiguity
    public function move(): string
    {
        return "Duck moves by swimming or flying";
    }
}

// The real diamond problem with TRAITS (PHP's multiple-behaviour tool)
trait A
{
    public function hello(): string { return "Hello from A"; }
}

trait B
{
    public function hello(): string { return "Hello from B"; }
}

class C
{
    use A, B {
        A::hello insteadof B;  // MUST explicitly resolve conflict
        B::hello as helloFromB; // alias for the other one
    }
}

$c = new C();
echo $c->hello();       // "Hello from A"
echo $c->helloFromB();  // "Hello from B"

// Composition avoids this entirely — no conflict possible
class Duck
{
    public function __construct(
        private Flyable   $flyBehavior,
        private Swimmable $swimBehavior
    ) {}

    public function fly(): string  { return $this->flyBehavior->move(); }
    public function swim(): string { return $this->swimBehavior->move(); }
    // Completely clear — no ambiguity
}
```

---

### 4.4 Liskov Substitution Principle (LSP)

> A child class must be **substitutable** for its parent class without breaking the program.

LSP is the test for whether inheritance is being used correctly.

```php
<?php

// Classic LSP violation — Rectangle / Square problem
class Rectangle
{
    protected int $width;
    protected int $height;

    public function setWidth(int $width): void   { $this->width  = $width; }
    public function setHeight(int $height): void { $this->height = $height; }
    public function getArea(): int               { return $this->width * $this->height; }
}

class Square extends Rectangle  // ❌ Square IS-A Rectangle — seems logical, but...
{
    // A square must always have equal sides
    public function setWidth(int $width): void
    {
        $this->width  = $width;
        $this->height = $width;  // override: sets both
    }

    public function setHeight(int $height): void
    {
        $this->width  = $height;
        $this->height = $height;  // override: sets both
    }
}

// This function expects a Rectangle and should work with any subclass
function testRectangle(Rectangle $rect): void
{
    $rect->setWidth(5);
    $rect->setHeight(10);
    // Expected area: 5 × 10 = 50
    assert($rect->getArea() === 50, "Area must be 50");
}

testRectangle(new Rectangle());  // ✅ passes — area = 50
testRectangle(new Square());     // ❌ FAILS — area = 100 (setHeight overwrote width)

// Square IS-A Rectangle mathematically — but NOT in code behavior
// Inheritance is wrong here — it VIOLATES LSP

// Solution: Composition / separate hierarchy
interface Shape
{
    public function getArea(): int;
}

class Rectangle implements Shape
{
    public function __construct(
        private int $width,
        private int $height
    ) {}

    public function getArea(): int { return $this->width * $this->height; }
}

class Square implements Shape
{
    public function __construct(private int $side) {}

    public function getArea(): int { return $this->side * $this->side; }
}

// Now they're separate — no broken substitution
function printArea(Shape $shape): void
{
    echo "Area: " . $shape->getArea() . "\n";
}

printArea(new Rectangle(5, 10));  // Area: 50 ✅
printArea(new Square(5));         // Area: 25 ✅
```

**LSP test:** If you need to write `if ($this instanceof Square)` in a method — inheritance is wrong. Use composition.

---

### 4.5 Composition Patterns

#### Strategy Pattern — Swap Algorithms at Runtime

```php
<?php

// The "how" is encapsulated in a strategy — swapped without changing context

interface SortStrategy
{
    public function sort(array &$data): void;
}

class BubbleSort implements SortStrategy
{
    public function sort(array &$data): void
    {
        $n = count($data);
        for ($i = 0; $i < $n - 1; $i++) {
            for ($j = 0; $j < $n - $i - 1; $j++) {
                if ($data[$j] > $data[$j + 1]) {
                    [$data[$j], $data[$j + 1]] = [$data[$j + 1], $data[$j]];
                }
            }
        }
    }
}

class QuickSort implements SortStrategy
{
    public function sort(array &$data): void
    {
        sort($data);  // simplified for example
    }
}

class MergeSort implements SortStrategy
{
    public function sort(array &$data): void
    {
        // merge sort implementation
        sort($data);  // simplified for example
    }
}

// Sorter HAS-A strategy — not IS-A BubbleSort
class Sorter
{
    private SortStrategy $strategy;

    public function __construct(SortStrategy $strategy)
    {
        $this->strategy = $strategy;
    }

    // Swap strategy at runtime — no subclass needed
    public function setStrategy(SortStrategy $strategy): void
    {
        $this->strategy = $strategy;
    }

    public function sort(array $data): array
    {
        $this->strategy->sort($data);
        return $data;
    }
}

$sorter = new Sorter(new BubbleSort());
$data   = [5, 2, 8, 1, 9, 3];
print_r($sorter->sort($data));

// Switch algorithm without creating new class
$sorter->setStrategy(new QuickSort());
print_r($sorter->sort($data));

// For small arrays: BubbleSort
// For large arrays: QuickSort
// For nearly-sorted: MergeSort
// Zero change to Sorter class — open/closed principle
```

---

#### Decorator Pattern — Add Behaviour Without Changing Class

```php
<?php

// Wrap behaviour around an object — stack decorators like LEGO

interface Logger
{
    public function log(string $message): void;
}

// Base implementation
class ConsoleLogger implements Logger
{
    public function log(string $message): void
    {
        echo $message . "\n";
    }
}

// Decorator 1 — adds timestamps
class TimestampLogger implements Logger
{
    public function __construct(private readonly Logger $inner) {}

    public function log(string $message): void
    {
        $this->inner->log("[" . date('Y-m-d H:i:s') . "] " . $message);
    }
}

// Decorator 2 — adds log levels
class LevelLogger implements Logger
{
    public function __construct(
        private readonly Logger $inner,
        private readonly string $level = 'INFO'
    ) {}

    public function log(string $message): void
    {
        $this->inner->log("[{$this->level}] " . $message);
    }
}

// Decorator 3 — filters by minimum level
class FilterLogger implements Logger
{
    private const LEVELS = ['DEBUG' => 0, 'INFO' => 1, 'WARN' => 2, 'ERROR' => 3];

    public function __construct(
        private readonly Logger $inner,
        private readonly string $minLevel = 'INFO'
    ) {}

    public function log(string $message): void
    {
        // Simple filter — pass through (real impl would check level)
        $this->inner->log($message);
    }
}

// Decorator 4 — writes to file additionally
class FileLogger implements Logger
{
    public function __construct(
        private readonly Logger $inner,
        private readonly string $path
    ) {}

    public function log(string $message): void
    {
        $this->inner->log($message);  // still log to console
        file_put_contents($this->path, $message . "\n", FILE_APPEND);
    }
}

// Stack decorators — compose exactly what you need
$logger = new FileLogger(
    new TimestampLogger(
        new LevelLogger(
            new ConsoleLogger(),
            'ERROR'
        )
    ),
    '/var/log/app.log'
);

$logger->log("Payment failed");
// Console: [2026-04-25 10:15:33] [ERROR] Payment failed
// File:    [2026-04-25 10:15:33] [ERROR] Payment failed

// Need a different combination? Just compose differently — zero new classes
$simpleLogger = new TimestampLogger(new ConsoleLogger());
$simpleLogger->log("User logged in");
// Console: [2026-04-25 10:15:33] User logged in
```

---

#### Mixin via Traits — PHP's Middle Ground

```php
<?php

// Traits let you inject reusable behaviour without inheritance
// This is composition-like — but with PHP's own mechanism

trait Timestampable
{
    private \DateTimeImmutable $createdAt;
    private \DateTimeImmutable $updatedAt;

    public function initTimestamps(): void
    {
        $this->createdAt = new \DateTimeImmutable();
        $this->updatedAt = new \DateTimeImmutable();
    }

    public function touch(): void
    {
        $this->updatedAt = new \DateTimeImmutable();
    }

    public function getCreatedAt(): \DateTimeImmutable { return $this->createdAt; }
    public function getUpdatedAt(): \DateTimeImmutable { return $this->updatedAt; }
}

trait SoftDeletable
{
    private ?\DateTimeImmutable $deletedAt = null;

    public function softDelete(): void
    {
        $this->deletedAt = new \DateTimeImmutable();
    }

    public function restore(): void
    {
        $this->deletedAt = null;
    }

    public function isDeleted(): bool
    {
        return $this->deletedAt !== null;
    }
}

trait Auditable
{
    private array $auditLog = [];

    public function audit(string $action, array $context = []): void
    {
        $this->auditLog[] = [
            'action'    => $action,
            'context'   => $context,
            'timestamp' => (new \DateTimeImmutable())->format('c'),
        ];
    }

    public function getAuditLog(): array { return $this->auditLog; }
}

// Mix in exactly what you need
class User
{
    use Timestampable, SoftDeletable, Auditable;

    public function __construct(
        private string $name,
        private string $email
    ) {
        $this->initTimestamps();
    }

    public function changeName(string $newName): void
    {
        $old         = $this->name;
        $this->name  = $newName;
        $this->touch();
        $this->audit('name_changed', ['from' => $old, 'to' => $newName]);
    }
}

class Product
{
    use Timestampable, SoftDeletable;  // No Auditable — not needed

    public function __construct(private string $name, private float $price)
    {
        $this->initTimestamps();
    }
}

$user = new User("Shivani", "shivani@gmail.com");
$user->changeName("Shivani Agarwal");
$user->softDelete();

echo $user->isDeleted() ? "Deleted\n" : "Active\n";  // Deleted
print_r($user->getAuditLog());
```

---

### 4.6 Dependency Injection as Composition

DI IS composition — you build objects by injecting their dependencies.

```php
<?php

// Full composition via DI Container — Symfony-style

interface CacheInterface
{
    public function get(string $key): mixed;
    public function set(string $key, mixed $value, int $ttl = 3600): void;
}

interface HttpClientInterface
{
    public function get(string $url): array;
}

interface LoggerInterface
{
    public function info(string $message, array $context = []): void;
    public function error(string $message, array $context = []): void;
}

// Concrete implementations
class RedisCache implements CacheInterface
{
    public function get(string $key): mixed    { /* Redis GET */ return null; }
    public function set(string $key, mixed $value, int $ttl = 3600): void { /* Redis SET */ }
}

class GuzzleHttpClient implements HttpClientInterface
{
    public function get(string $url): array    { /* Guzzle HTTP GET */ return []; }
}

class MonologLogger implements LoggerInterface
{
    public function info(string $msg, array $ctx = []): void  { /* Monolog INFO */ }
    public function error(string $msg, array $ctx = []): void { /* Monolog ERROR */ }
}

// Service composed entirely of injected collaborators
// It has NO knowledge of HOW caching/http/logging works
class WeatherService
{
    public function __construct(
        private readonly CacheInterface      $cache,
        private readonly HttpClientInterface $http,
        private readonly LoggerInterface     $logger
    ) {}

    public function getWeather(string $city): array
    {
        $cacheKey = "weather_{$city}";
        $cached   = $this->cache->get($cacheKey);

        if ($cached !== null) {
            $this->logger->info("Cache hit", ['city' => $city]);
            return $cached;
        }

        $this->logger->info("Fetching weather", ['city' => $city]);

        try {
            $data = $this->http->get("https://api.weather.com/{$city}");
            $this->cache->set($cacheKey, $data, 1800);
            return $data;
        } catch (\Exception $e) {
            $this->logger->error("Weather fetch failed", [
                'city'  => $city,
                'error' => $e->getMessage()
            ]);
            throw $e;
        }
    }
}

// Compose real service
$weatherService = new WeatherService(
    new RedisCache(),
    new GuzzleHttpClient(),
    new MonologLogger()
);

// Compose test service — same class, different collaborators
class ArrayCache implements CacheInterface
{
    private array $store = [];
    public function get(string $key): mixed    { return $this->store[$key] ?? null; }
    public function set(string $key, mixed $value, int $ttl = 3600): void { $this->store[$key] = $value; }
}

class MockHttpClient implements HttpClientInterface
{
    public function get(string $url): array { return ['temp' => 22, 'city' => 'Berlin']; }
}

class NullLogger implements LoggerInterface
{
    public function info(string $msg, array $ctx = []): void  {}
    public function error(string $msg, array $ctx = []): void {}
}

$testService = new WeatherService(
    new ArrayCache(),       // in-memory cache
    new MockHttpClient(),   // fake HTTP
    new NullLogger()        // silent logger
);

// WeatherService code didn't change AT ALL — only what was injected
```

---

### 4.7 When Inheritance IS Correct

Not everything should be composition. Inheritance is right when:

```php
<?php

// ✅ CORRECT inheritance — true IS-A, stable hierarchy, LSP respected

// Exception hierarchies — natural IS-A
class AppException extends \RuntimeException {}
class ValidationException extends AppException
{
    public function __construct(private array $errors)
    {
        parent::__construct("Validation failed: " . implode(', ', $errors));
    }
    public function getErrors(): array { return $this->errors; }
}
class DatabaseException extends AppException {}
class NotFoundException extends DatabaseException
{
    public function __construct(string $resource, int|string $id)
    {
        parent::__construct("$resource with ID '$id' not found");
    }
}

// ✅ Framework base classes — controlled hierarchy
// Symfony: Controller, Command, Event, Constraint
// Laravel: Model, Controller, Job, Notification

// ✅ Template Method Pattern — skeleton fixed, steps overridable
abstract class DataImporter
{
    // Template method — final: subclasses cannot change the algorithm
    final public function import(string $source): ImportResult
    {
        $raw       = $this->readSource($source);   // subclass implements
        $validated = $this->validate($raw);         // shared
        $mapped    = $this->mapToEntities($mapped ?? $validated); // subclass
        $saved     = $this->persist($mapped);       // shared
        $this->onComplete($saved);                  // hook — optional override
        return new ImportResult($saved);
    }

    abstract protected function readSource(string $source): array;
    abstract protected function mapToEntities(array $raw): array;

    protected function validate(array $data): array
    {
        return array_filter($data, fn($row) => !empty($row));
    }

    protected function persist(array $entities): array
    {
        // default DB persist — can be overridden
        return $entities;
    }

    protected function onComplete(array $saved): void {} // optional hook
}

class CsvImporter extends JsonImporter
{
    protected function readSource(string $path): array
    {
        return array_map('str_getcsv', file($path));
    }

    protected function mapToEntities(array $raw): array
    {
        return array_map(fn($row) => ['name' => $row[0], 'email' => $row[1]], $raw);
    }
}

// These ARE correct uses of inheritance
```

**Inheritance IS right when:**

- The IS-A relationship is genuinely true AND stable
- LSP is not violated — child can fully substitute parent
- You are defining a template/skeleton algorithm (Template Method)
- Exception hierarchies — natural, stable, correct IS-A
- Framework base classes where the framework author controls the hierarchy

---

### 4.8 Deep Real-World Refactor — Inheritance → Composition

**Scenario:** An e-commerce order processing system originally built with inheritance.

```php
<?php
// ============================================================
// BEFORE — Inheritance-based (fragile, hard to extend) ❌
// ============================================================

class OrderProcessor
{
    public function process(array $order): void
    {
        $this->validate($order);
        $this->calculateTax($order);
        $this->applyDiscount($order);
        $this->charge($order);
        $this->sendConfirmation($order);
    }

    protected function validate(array $order): void
    {
        if (empty($order['items'])) {
            throw new \InvalidArgumentException("Order has no items");
        }
    }

    protected function calculateTax(array &$order): void
    {
        $order['tax'] = $order['subtotal'] * 0.19;
    }

    protected function applyDiscount(array &$order): void
    {
        // no discount by default
    }

    protected function charge(array $order): void
    {
        echo "Charging €{$order['total']} via default payment\n";
    }

    protected function sendConfirmation(array $order): void
    {
        echo "Sending email confirmation\n";
    }
}

// To add discount logic: extend the class
class DiscountedOrderProcessor extends OrderProcessor
{
    protected function applyDiscount(array &$order): void
    {
        $order['discount'] = $order['subtotal'] * 0.1;
        $order['total']    = $order['subtotal'] - $order['discount'] + $order['tax'];
    }
}

// To add Stripe payment: extend again
class StripeOrderProcessor extends DiscountedOrderProcessor
{
    protected function charge(array $order): void
    {
        echo "Charging via Stripe: €{$order['total']}\n";
    }
}

// To add Slack notification: extend again
class SlackNotifyingStripeOrderProcessor extends StripeOrderProcessor
{
    protected function sendConfirmation(array $order): void
    {
        parent::sendConfirmation($order);
        echo "Sending Slack notification\n";
    }
}

// Class explosion! And we haven't even started:
// PayPalDiscountedSlackNotifyingOrderProcessor?
// StripeNoDiscountPushNotifyingOrderProcessor?
// The inheritance tree becomes unmanageable
```

```php
<?php
// ============================================================
// AFTER — Composition-based (flexible, extensible, testable) ✅
// ============================================================

// Interfaces — clear contracts
interface OrderValidator
{
    public function validate(array $order): void;
}

interface TaxCalculator
{
    public function calculate(array &$order): void;
}

interface DiscountApplier
{
    public function apply(array &$order): void;
}

interface PaymentProcessor
{
    public function charge(array $order): void;
}

interface NotificationSender
{
    public function send(array $order): void;
}

// -------- Validators --------
class BasicOrderValidator implements OrderValidator
{
    public function validate(array $order): void
    {
        if (empty($order['items'])) {
            throw new \InvalidArgumentException("Order has no items");
        }
        if (($order['subtotal'] ?? 0) <= 0) {
            throw new \InvalidArgumentException("Order subtotal must be positive");
        }
    }
}

// -------- Tax Calculators --------
class GermanTaxCalculator implements TaxCalculator
{
    public function calculate(array &$order): void
    {
        $order['tax']   = $order['subtotal'] * 0.19;
        $order['total'] = $order['subtotal'] + $order['tax'];
    }
}

class UKTaxCalculator implements TaxCalculator
{
    public function calculate(array &$order): void
    {
        $order['tax']   = $order['subtotal'] * 0.20;
        $order['total'] = $order['subtotal'] + $order['tax'];
    }
}

// -------- Discount Appliers --------
class NoDiscount implements DiscountApplier
{
    public function apply(array &$order): void {} // no-op
}

class PercentageDiscount implements DiscountApplier
{
    public function __construct(private float $percent) {}

    public function apply(array &$order): void
    {
        $order['discount'] = $order['subtotal'] * ($this->percent / 100);
        $order['subtotal'] -= $order['discount'];
    }
}

class VipDiscount implements DiscountApplier
{
    public function apply(array &$order): void
    {
        $order['discount'] = $order['subtotal'] * 0.15;
        $order['subtotal'] -= $order['discount'];
    }
}

// -------- Payment Processors --------
class StripePayment implements PaymentProcessor
{
    public function charge(array $order): void
    {
        echo "💳 Stripe: charging €{$order['total']}\n";
    }
}

class PayPalPayment implements PaymentProcessor
{
    public function charge(array $order): void
    {
        echo "💰 PayPal: charging €{$order['total']}\n";
    }
}

// -------- Notification Senders --------
class EmailNotification implements NotificationSender
{
    public function send(array $order): void
    {
        echo "📧 Email confirmation sent for order #{$order['id']}\n";
    }
}

class SmsNotification implements NotificationSender
{
    public function send(array $order): void
    {
        echo "📱 SMS confirmation sent for order #{$order['id']}\n";
    }
}

class SlackNotification implements NotificationSender
{
    public function send(array $order): void
    {
        echo "💬 Slack notification sent for order #{$order['id']}\n";
    }
}

// Multi-notification sender — composing senders
class CompositeNotification implements NotificationSender
{
    /** @var NotificationSender[] */
    private array $senders;

    public function __construct(NotificationSender ...$senders)
    {
        $this->senders = $senders;
    }

    public function send(array $order): void
    {
        foreach ($this->senders as $sender) {
            $sender->send($order);
        }
    }
}

// -------- The Processor — pure composition --------
final class OrderProcessor
{
    public function __construct(
        private readonly OrderValidator    $validator,
        private readonly DiscountApplier   $discount,
        private readonly TaxCalculator     $tax,
        private readonly PaymentProcessor  $payment,
        private readonly NotificationSender $notification
    ) {}

    public function process(array $order): void
    {
        $this->validator->validate($order);
        $this->discount->apply($order);
        $this->tax->calculate($order);
        $this->payment->charge($order);
        $this->notification->send($order);
    }
}

// ============================================================
// Compose any combination — ZERO new classes needed
// ============================================================

// Standard German customer, Stripe, email only
$standardProcessor = new OrderProcessor(
    validator:    new BasicOrderValidator(),
    discount:     new NoDiscount(),
    tax:          new GermanTaxCalculator(),
    payment:      new StripePayment(),
    notification: new EmailNotification()
);

// VIP customer, PayPal, email + SMS + Slack
$vipProcessor = new OrderProcessor(
    validator:    new BasicOrderValidator(),
    discount:     new VipDiscount(),
    tax:          new GermanTaxCalculator(),
    payment:      new PayPalPayment(),
    notification: new CompositeNotification(
        new EmailNotification(),
        new SmsNotification(),
        new SlackNotification()
    )
);

// UK customer, 10% promo, Stripe, email only
$ukPromoProcessor = new OrderProcessor(
    validator:    new BasicOrderValidator(),
    discount:     new PercentageDiscount(10),
    tax:          new UKTaxCalculator(),
    payment:      new StripePayment(),
    notification: new EmailNotification()
);

// The combinations are infinite — no class explosion
// Each piece is independently testable
// OrderProcessor never changes — open/closed principle
$order = ['id' => 'ORD-001', 'items' => ['item1'], 'subtotal' => 100.0];
$vipProcessor->process($order);
```

---

## 5. For the Critics 🔥

### Criticism 1 — "Favour Composition" Is Cargo-Culted Without Understanding

**The Argument:**

> "Every junior developer now avoids inheritance entirely — replacing simple, readable `class Dog extends Animal` with a labyrinth of interfaces, factories, and injected dependencies. The cure is worse than the disease. A `OrderProcessor` class with five injected interfaces is harder to understand than a simple class hierarchy."

**Valid — when taken to extremes.**
Composition is not about adding complexity. It is about adding flexibility at the RIGHT seams. If your `Dog` truly IS an `Animal` and always will be — extend it. If your `OrderProcessor` needs to swap payment providers — compose it. The failure mode is applying composition everywhere dogmatically, which creates interface explosion. The rule is: compose where variation IS EXPECTED, inherit where the relationship IS STABLE.

---

### Criticism 2 — Inheritance is Not Dead — Frameworks Prove It

**The Argument:**

> "Symfony's Controller, Laravel's Model, Eloquent's relationships — all inheritance-heavy. The most successful PHP frameworks in the world use inheritance as a central design tool. If it was so bad, why do they keep using it?"

**Correct — with important context.**
Framework base classes are controlled inheritance — the framework author knows the hierarchy will never change. When you extend Symfony's `AbstractController`, you're using a stable, well-designed IS-A relationship. The fragile base class problem matters when you are BOTH the base class author AND the child class author simultaneously. Frameworks have solved this by freezing their base classes carefully. The criticism is valid: inheritance is not dead. The guidance is use it where IT WAS DESIGNED TO BE USED.

---

### Criticism 3 — Composition Creates "Interface Hell"

**The Argument:**

> "Composing everything creates: 1 interface for every behaviour, 1 concrete class per implementation, 1 factory or container to wire it all up, 1 mock per interface for testing. A simple feature that took 1 class with inheritance now takes 6 files. This is not clarity — it is ceremony."

**Partially valid.**
This is the cost of full composition — upfront investment in abstractions. The payoff is flexibility and testability later. In a codebase that never changes requirements, composition's overhead is pure cost. In real production systems where requirements change constantly (new payment provider, new notification channel) — the investment pays back 10x. The discipline is: **only introduce an abstraction when variation is NEEDED or EXPECTED**. `OrderProcessor` with swappable payment? Compose. `Dog` that only ever barks? Just extend `Animal`.

---

### Criticism 4 — PHP Traits Are Neither Composition Nor Inheritance

**The Argument:**

> "PHP Traits are copy-paste at compile time. They don't create real IS-A or HAS-A relationships. They create invisible dependencies, conflict problems, and shared state nightmares. `use Timestampable` looks like composition but behaves like code injection. They are a design smell masquerading as a feature."

**Largely valid.**
Traits ARE PHP's acknowledged workaround for not having mixin support. They create tight coupling (trait accesses class properties it doesn't own), conflict resolution is verbose (`insteadof`), and they make code harder to static-analyze. The correct use of traits is narrow: pure behaviour with no state access into the host class — essentially utility methods. When traits access `$this->property` — they've crossed into dangerous territory. Real composition (injected collaborators) is almost always superior to trait overuse.

---

### Criticism 5 — The "Fragile Base Class" Is an Engineering Discipline Problem

**The Argument:**

> "The fragile base class problem is solved by: good documentation, semantic versioning, not changing base class behavior. It's not a language or design problem — it's a team discipline problem. Blaming inheritance for teams that don't follow contracts is like blaming knives for kitchen injuries."

**Correct as a pure argument — but impractical at scale.**
Yes — if every base class author perfectly documented every assumption, never changed internal behavior, and versioned every change — the fragile base class problem disappears. In practice: teams grow, authors leave, documentation rots, and "I'll just quickly optimize this protected method" happens every week. Composition removes the POSSIBILITY of the problem — not just the likelihood. Engineering systems should account for human fallibility, not require perfect discipline. Composition is the defensive design choice.

---

## 6. Full Comparison Table

|                           | Inheritance                                | Composition                              |
| ------------------------- | ------------------------------------------ | ---------------------------------------- |
| **Relationship**          | IS-A                                       | HAS-A                                    |
| **Coupling**              | Tight — child depends on parent internals  | Loose — depends on interface only        |
| **Flexibility**           | Low — hierarchy fixed at class definition  | High — behavior injected at construction |
| **Code reuse**            | Via extending — automatic                  | Via delegation — explicit                |
| **Multiple parents**      | ❌ PHP allows only one                     | ✅ Compose as many as needed             |
| **Testing**               | Harder — cannot mock parent behavior       | Easy — swap real for mock                |
| **Encapsulation**         | Often broken — protected exposes internals | Strong — only interface visible          |
| **LSP**                   | Must be respected — easy to violate        | Not applicable — no substitution         |
| **Fragile Base Class**    | Real risk                                  | Not possible                             |
| **Gorilla-Banana**        | Common problem                             | Not possible                             |
| **Diamond problem**       | PHP blocks it — traits have conflicts      | Not possible                             |
| **Runtime behavior swap** | ❌ Not possible — fixed at definition      | ✅ Easy — inject different impl          |
| **Readability (simple)**  | ✅ More obvious                            | Sometimes verbose                        |
| **Readability (complex)** | ❌ Deep hierarchies are hard to follow     | ✅ Flat, explicit                        |
| **Best for**              | Stable IS-A, Template Method, Exceptions   | Almost everything else                   |

---

## 7. Decision Guide

### Should I use Inheritance or Composition?

```
Ask: "Does Class B genuinely IS-A Class A?"
│
├── YES — and this will NEVER change?
│   └── Ask: "Does LSP hold — can B fully substitute A?"
│       ├── YES → Inheritance is appropriate
│       └── NO  → Use composition — the IS-A is mathematically true but not behaviorally
│
└── NO — B just USES some of A's behavior?
    └── Composition — always

Quick tests for inheritance being WRONG:
  - You wrote: if ($this instanceof ChildClass) in parent → WRONG
  - Child overrides method to throw exception → WRONG (LSP violation)
  - You're inheriting just to reuse one method → WRONG (use trait or injection)
  - You need to inherit from two classes → WRONG (use composition)
  - The parent class has methods the child never uses → WRONG (Gorilla-Banana)
```

### Which Composition Pattern?

```
Do you need to SWAP the behaviour at runtime?
  └── YES → Strategy Pattern (inject the strategy)

Do you need to ADD behaviour around an existing class?
  └── YES → Decorator Pattern (wrap the collaborator)

Do you need to share behaviour across UNRELATED classes (no common parent)?
  └── YES → Trait (PHP mixin — with caution, no state access)

Do you need to INJECT dependencies for testability?
  └── YES → Dependency Injection (compose via constructor)

Do you need one interface to control MULTIPLE implementations?
  └── YES → Composite Pattern (collection of same interface)
```

### One-Line Rules

```
1. Start with composition — switch to inheritance if IS-A is provably true and stable
2. If the hierarchy is more than 2 levels deep — stop and reconsider
3. If a child class overrides a method to do nothing or throw — it's wrong inheritance
4. If you cannot swap a dependency in a test — you need more composition
5. Use traits for stateless behaviour sharing only
6. Prefer interfaces over abstract classes — unless shared implementation is the point
```

---

## Final One-Liner Summary

| Audience         | Summary                                                                                                                                                                                                                                                                                                                                                                              |
| ---------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| 🧸 5-year-old    | Inheritance = copying the whole LEGO box. Composition = picking only the pieces you need. Most of the time, picking is better.                                                                                                                                                                                                                                                       |
| 💻 Mid-level dev | Inheritance shares code through IS-A. Composition shares behaviour through HAS-A. Use inheritance for true IS-A relationships. Use composition when you need to swap, test, or combine behaviours.                                                                                                                                                                                   |
| 🧠 Expert dev    | Fragile Base Class, Gorilla-Banana, and LSP violations are the cost of wrong inheritance. Strategy, Decorator, DI, and Traits are the tools of composition. The real skill is knowing WHEN each is correct — not applying one dogmatically.                                                                                                                                          |
| 🔥 Critics       | "Favour composition" is cargo-culted into interface hell by developers who don't understand it. Frameworks prove inheritance is not dead. Traits are neither properly. The discipline problem of fragile base classes is real but not solely a language fault. The answer is not one or the other — it is understanding the tradeoffs well enough to make the right call every time. |

---

_Happy Designing! 🏗️🚀_
