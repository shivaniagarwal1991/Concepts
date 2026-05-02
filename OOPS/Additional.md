# Additional OOP Concepts Every PHP Developer Must Know

### SOLID · Magic Methods · Object Cloning · Object Comparison · Traits · Anonymous Classes

#### 5-Year-Old 🧸 → Mid-Level 💻 → Expert 🧠 → Critics 🔥

---

## Table of Contents

1. [SOLID Principles](#1-solid-principles)
2. [Magic Methods](#2-magic-methods)
3. [Object Cloning](#3-object-cloning)
4. [Object Comparison — == vs ===](#4-object-comparison----vs-)
5. [Traits — PHP Mixins](#5-traits--php-mixins)
6. [Anonymous Classes](#6-anonymous-classes)
7. [Object Iteration](#7-object-iteration)
8. [Late Static Binding](#8-late-static-binding)

---

## 1. SOLID Principles

SOLID = 5 rules that make OOP code maintainable, flexible, and testable.

### 5-Year-Old Version 🧸

```
S — Single Responsibility: One box does ONE job. Don't put your toys AND your food in the same box.
O — Open/Closed:           You can ADD new LEGO pieces. You shouldn't BREAK existing ones.
L — Liskov Substitution:   If your toy car fits the toy road — a toy TRUCK should also fit.
I — Interface Segregation: Don't force a fish to have a "fly" button it will never use.
D — Dependency Inversion:  The TV remote should work with ANY TV — not be glued to one specific TV.
```

---

### S — Single Responsibility Principle

> A class should have **only one reason to change**.

```php
<?php

// ❌ WRONG — one class doing too many jobs
class User
{
    public string $name;
    public string $email;

    // Job 1: user data
    public function getName(): string { return $this->name; }

    // Job 2: database — should NOT be here
    public function saveToDatabase(): void
    {
        // SQL logic here — database reason to change
        echo "Saving {$this->name} to DB\n";
    }

    // Job 3: email — should NOT be here
    public function sendWelcomeEmail(): void
    {
        // SMTP logic here — email reason to change
        echo "Sending welcome to {$this->email}\n";
    }

    // Job 4: report — should NOT be here
    public function generateReport(): string
    {
        return "User report for {$this->name}";
    }
}
// If DB changes, email changes, OR report format changes — this class changes
// Three reasons to change = three responsibilities = SRP violation

// ✅ CORRECT — each class has ONE job
class User
{
    public function __construct(
        public readonly string $name,
        public readonly string $email
    ) {}
}

class UserRepository           // handles DB
{
    public function save(User $user): void
    {
        echo "Saving {$user->name} to DB\n";
    }
    public function findById(int $id): ?User { return null; }
}

class UserMailer               // handles email
{
    public function sendWelcome(User $user): void
    {
        echo "Sending welcome to {$user->email}\n";
    }
}

class UserReportGenerator      // handles reports
{
    public function generate(User $user): string
    {
        return "User report for {$user->name}";
    }
}

// Now: DB change → only UserRepository changes
//      Email change → only UserMailer changes
//      Report change → only UserReportGenerator changes
```

---

### O — Open/Closed Principle

> A class should be **open for extension**, **closed for modification**.

```php
<?php

// ❌ WRONG — must modify class to add new discount type
class DiscountCalculator
{
    public function calculate(string $type, float $price): float
    {
        if ($type === 'percentage') {
            return $price * 0.9;         // 10% off
        } elseif ($type === 'fixed') {
            return $price - 10.0;         // €10 off
        } elseif ($type === 'bogo') {
            return $price * 0.5;          // buy-one-get-one
        }
        // Adding 'flash_sale' → MODIFY this class → violates OCP
        return $price;
    }
}

// ✅ CORRECT — add new type by extending, not modifying
interface DiscountStrategy
{
    public function apply(float $price): float;
}

class PercentageDiscount implements DiscountStrategy
{
    public function __construct(private float $percent) {}
    public function apply(float $price): float { return $price * (1 - $this->percent / 100); }
}

class FixedDiscount implements DiscountStrategy
{
    public function __construct(private float $amount) {}
    public function apply(float $price): float { return max(0, $price - $this->amount); }
}

class BuyOneGetOneDiscount implements DiscountStrategy
{
    public function apply(float $price): float { return $price * 0.5; }
}

// Adding new type: create new class — ZERO existing code changes
class FlashSaleDiscount implements DiscountStrategy
{
    public function apply(float $price): float { return $price * 0.7; } // 30% off
}

class PriceCalculator
{
    public function calculate(float $price, DiscountStrategy $discount): float
    {
        return $discount->apply($price);
    }
}

$calc = new PriceCalculator();
echo $calc->calculate(100.0, new PercentageDiscount(10));  // 90.0
echo $calc->calculate(100.0, new FixedDiscount(15));       // 85.0
echo $calc->calculate(100.0, new FlashSaleDiscount());     // 70.0
```

---

### L — Liskov Substitution Principle

> Subclasses must be **fully substitutable** for their parent class.

```php
<?php

// ❌ Classic LSP violation
class Bird
{
    public function fly(): string { return "Flying!"; }
}

class Penguin extends Bird
{
    public function fly(): string
    {
        throw new \RuntimeException("Penguins can't fly!"); // ❌ violates LSP
    }
}

function makeBirdFly(Bird $bird): void
{
    echo $bird->fly() . "\n"; // Crashes for Penguin — substitution broken
}

makeBirdFly(new Bird());    // ✅ "Flying!"
makeBirdFly(new Penguin()); // ❌ RuntimeException

// ✅ Correct hierarchy — separate what can and cannot fly
interface Bird       { public function eat(): string; }
interface FlyingBird extends Bird { public function fly(): string; }

class Sparrow implements FlyingBird
{
    public function eat(): string { return "Sparrow eats seeds"; }
    public function fly(): string { return "Sparrow flies!"; }
}

class Penguin implements Bird   // Does NOT implement FlyingBird
{
    public function eat(): string { return "Penguin eats fish"; }
    public function swim(): string { return "Penguin swims!"; }
}

function makeFlyingBirdFly(FlyingBird $bird): void
{
    echo $bird->fly() . "\n"; // Only flying birds can be passed
}

makeFlyingBirdFly(new Sparrow());   // ✅
// makeFlyingBirdFly(new Penguin()); // ❌ Type error at compile time — caught early!
```

---

### I — Interface Segregation Principle

> A class should **not be forced** to implement methods it does not need.

```php
<?php

// ❌ Fat interface — forces all classes to implement everything
interface Worker
{
    public function work(): void;
    public function eat(): void;
    public function sleep(): void;
    public function takeBreak(): void;
}

class Robot implements Worker
{
    public function work(): void      { echo "Robot working\n"; }
    public function eat(): void       { throw new \RuntimeException("Robots don't eat!"); } // ❌
    public function sleep(): void     { throw new \RuntimeException("Robots don't sleep!"); } // ❌
    public function takeBreak(): void { throw new \RuntimeException("Robots don't break!"); } // ❌
}

// ✅ Segregated interfaces — each class implements only what it needs
interface Workable   { public function work(): void; }
interface Eatable    { public function eat(): void; }
interface Sleepable  { public function sleep(): void; }
interface Breakable  { public function takeBreak(): void; }

class HumanWorker implements Workable, Eatable, Sleepable, Breakable
{
    public function work(): void      { echo "Human working\n"; }
    public function eat(): void       { echo "Human eating\n"; }
    public function sleep(): void     { echo "Human sleeping\n"; }
    public function takeBreak(): void { echo "Human on break\n"; }
}

class Robot implements Workable   // ONLY what robots actually do
{
    public function work(): void   { echo "Robot working 24/7\n"; }
}

// Functions only require the interfaces they actually use
function startWorkDay(Workable $worker): void
{
    $worker->work();
}

function lunchTime(Eatable $worker): void
{
    $worker->eat();
}

startWorkDay(new Robot());     // ✅
startWorkDay(new HumanWorker()); // ✅
lunchTime(new HumanWorker());    // ✅
// lunchTime(new Robot());       // ❌ Type error — Robot is not Eatable — correct!
```

---

### D — Dependency Inversion Principle

> High-level modules should **not depend on low-level modules**. Both should depend on **abstractions**.

```php
<?php

// ❌ High-level class depends on low-level (concrete) class
class OrderService
{
    private MySQLDatabase $db;      // tightly coupled to MySQL
    private SmtpMailer    $mailer;  // tightly coupled to SMTP

    public function __construct()
    {
        $this->db     = new MySQLDatabase();   // creates its own dependency ❌
        $this->mailer = new SmtpMailer();      // creates its own dependency ❌
    }

    public function placeOrder(array $items): void
    {
        $this->db->save(['items' => $items]);
        $this->mailer->send("Order confirmed!");
    }
}

// ✅ CORRECT — depend on abstractions, inject concretions
interface DatabaseInterface { public function save(array $data): void; }
interface MailerInterface   { public function send(string $message): void; }

class MySQLDatabase implements DatabaseInterface
{
    public function save(array $data): void { echo "MySQL: saved\n"; }
}

class PostgresDatabase implements DatabaseInterface
{
    public function save(array $data): void { echo "Postgres: saved\n"; }
}

class SmtpMailer implements MailerInterface
{
    public function send(string $message): void { echo "SMTP: $message\n"; }
}

class LogMailer implements MailerInterface
{
    public function send(string $message): void { echo "[LOG] $message\n"; }
}

class OrderService
{
    // Depends on ABSTRACTIONS — not concrete classes
    public function __construct(
        private DatabaseInterface $db,
        private MailerInterface   $mailer
    ) {}

    public function placeOrder(array $items): void
    {
        $this->db->save(['items' => $items]);
        $this->mailer->send("Order confirmed!");
    }
}

// Production
$service = new OrderService(new MySQLDatabase(), new SmtpMailer());

// Testing — swap without touching OrderService
$service = new OrderService(
    new class implements DatabaseInterface { public function save(array $d): void {} },
    new class implements MailerInterface   { public function send(string $m): void {} }
);

// Switch to Postgres + Log mailer
$service = new OrderService(new PostgresDatabase(), new LogMailer());
```

---

## 2. Magic Methods

Magic methods are automatically called by PHP in specific situations.

### Complete Reference

```php
<?php

class MagicDemo
{
    private array $data = [];

    // ── Lifecycle ─────────────────────────────────────────────────
    public function __construct(array $initial = [])
    {
        $this->data = $initial;
        echo "__construct called\n";
    }

    public function __destruct()
    {
        echo "__destruct called — object cleaned up\n";
    }

    // ── Property Access ───────────────────────────────────────────
    public function __get(string $name): mixed
    {
        echo "__get($name)\n";
        return $this->data[$name] ?? null;
    }

    public function __set(string $name, mixed $value): void
    {
        echo "__set($name, $value)\n";
        $this->data[$name] = $value;
    }

    public function __isset(string $name): bool
    {
        echo "__isset($name)\n";
        return isset($this->data[$name]);
    }

    public function __unset(string $name): void
    {
        echo "__unset($name)\n";
        unset($this->data[$name]);
    }

    // ── String Conversion ─────────────────────────────────────────
    public function __toString(): string
    {
        return json_encode($this->data);
    }

    // ── Invocation ────────────────────────────────────────────────
    public function __invoke(string $message): string
    {
        return "Invoked with: $message";
    }

    // ── Method Interception ───────────────────────────────────────
    public function __call(string $name, array $args): mixed
    {
        echo "__call($name, " . implode(', ', $args) . ")\n";
        return null;
    }

    public static function __callStatic(string $name, array $args): mixed
    {
        echo "__callStatic($name)\n";
        return null;
    }

    // ── Cloning ───────────────────────────────────────────────────
    public function __clone(): void
    {
        // Deep copy nested objects here
        echo "__clone called\n";
    }

    // ── Serialization ─────────────────────────────────────────────
    public function __serialize(): array
    {
        return ['data' => $this->data];
    }

    public function __unserialize(array $data): void
    {
        $this->data = $data['data'];
    }

    // ── Debugging ─────────────────────────────────────────────────
    public function __debugInfo(): array
    {
        // Control what var_dump() shows
        return ['count' => count($this->data), 'keys' => array_keys($this->data)];
    }
}

$obj = new MagicDemo(['name' => 'Shivani']);

// Property access magic
echo $obj->name;           // __get(name) → "Shivani"
$obj->age   = 30;          // __set(age, 30)
isset($obj->age);          // __isset(age) → true
unset($obj->age);          // __unset(age)

// String conversion
echo (string) $obj;        // __toString → {"name":"Shivani"}
echo $obj;                 // Same — PHP auto-calls __toString

// Invocation
echo $obj("Hello!");       // __invoke → "Invoked with: Hello!"

// Undefined method
$obj->unknownMethod(1, 2); // __call(unknownMethod, [1, 2])
MagicDemo::staticMethod(); // __callStatic(staticMethod)

// Serialization
$serialized = serialize($obj);       // __serialize called
$restored   = unserialize($serialized); // __unserialize called
```

---

## 3. Object Cloning

By default, PHP creates a **shallow copy** on `clone`. This means nested objects are SHARED, not copied.

### Shallow vs Deep Clone

```php
<?php

class Address
{
    public function __construct(
        public string $street,
        public string $city
    ) {}
}

class User
{
    public function __construct(
        public string  $name,
        public Address $address   // object — reference by default
    ) {}
}

// ── Shallow Clone (default) ──────────────────────────────────────
$user1 = new User("Shivani", new Address("Unter den Linden 1", "Berlin"));
$user2 = clone $user1;  // shallow copy

$user2->name           = "Alex";           // ✅ independent scalar
$user2->address->city  = "Munich";         // ❌ SHARED — also changes user1!

echo $user1->name;          // "Shivani" ✅
echo $user1->address->city; // "Munich" ❌ — should still be "Berlin"!

// ── Deep Clone (via __clone) ──────────────────────────────────────
class User
{
    public function __construct(
        public string  $name,
        public Address $address
    ) {}

    public function __clone(): void
    {
        // Deep copy — clone the nested object too
        $this->address = clone $this->address;
    }
}

$user1 = new User("Shivani", new Address("Unter den Linden 1", "Berlin"));
$user2 = clone $user1;

$user2->name           = "Alex";
$user2->address->city  = "Munich";

echo $user1->name;          // "Shivani" ✅
echo $user1->address->city; // "Berlin"  ✅ — completely independent now!
echo $user2->address->city; // "Munich"  ✅

// ── Immutable pattern — returns cloned modified version ──────────
class Config
{
    private array $settings;

    public function __construct(array $settings = [])
    {
        $this->settings = $settings;
    }

    public function with(string $key, mixed $value): static
    {
        $clone = clone $this;
        $clone->settings[$key] = $value;
        return $clone;
    }

    public function get(string $key, mixed $default = null): mixed
    {
        return $this->settings[$key] ?? $default;
    }
}

$base    = new Config(['debug' => false, 'env' => 'production']);
$dev     = $base->with('debug', true)->with('env', 'development');
$staging = $base->with('env', 'staging');

echo $base->get('debug');    // false ← original unchanged
echo $dev->get('debug');     // true
echo $staging->get('env');   // staging
```

---

## 4. Object Comparison — `==` vs `===`

```php
<?php

class Point
{
    public function __construct(
        public int $x,
        public int $y
    ) {}
}

$p1 = new Point(3, 4);
$p2 = new Point(3, 4);  // same values, DIFFERENT instance
$p3 = $p1;              // same instance

// == (loose) — compares property VALUES
var_dump($p1 == $p2);   // true  — same property values
var_dump($p1 == $p3);   // true  — same instance, same values

// === (strict) — compares IDENTITY (same object in memory)
var_dump($p1 === $p2);  // false — different instances!
var_dump($p1 === $p3);  // true  — same instance

// ── Implementing value equality ────────────────────────────────
final class Money
{
    public function __construct(
        private readonly int    $amount,
        private readonly string $currency
    ) {}

    // Explicit equality method — don't rely on ==
    public function equals(self $other): bool
    {
        return $this->amount   === $other->amount
            && $this->currency === $other->currency;
    }

    public function getAmount(): int    { return $this->amount; }
    public function getCurrency(): string { return $this->currency; }
}

$price1 = new Money(999, 'EUR');
$price2 = new Money(999, 'EUR');
$price3 = new Money(999, 'USD');

var_dump($price1 == $price2);          // true  (== on objects compares properties)
var_dump($price1 === $price2);         // false (different instances)
var_dump($price1->equals($price2));    // true  ✅ explicit value equality
var_dump($price1->equals($price3));    // false ✅ different currency

// ── Why == on objects is dangerous ────────────────────────────
class UserEntity
{
    public function __construct(
        public int    $id,
        public string $name,
        public string $email
    ) {}
}

$u1 = new UserEntity(1, "Shivani", "s@gmail.com");
$u2 = new UserEntity(1, "Shivani Updated", "s@gmail.com"); // same ID, different name

var_dump($u1 == $u2);  // FALSE — different name → == says they're different
// But they are the SAME user in the database (same ID)!
// For entities: identity = ID, not property values

class UserEntity
{
    public function __construct(
        public int    $id,
        public string $name,
        public string $email
    ) {}

    public function isSameUser(self $other): bool
    {
        return $this->id === $other->id; // identity based on ID, not values
    }
}
```

---

## 5. Traits — PHP Mixins

Traits let you reuse code across classes that don't share a common ancestor.

### Basic Traits

```php
<?php

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

    public function restore(): void     { $this->deletedAt = null; }
    public function isDeleted(): bool   { return $this->deletedAt !== null; }
}

trait HasUuid
{
    private string $uuid;

    public function initUuid(): void
    {
        $this->uuid = sprintf('%04x%04x-%04x-%04x-%04x-%04x%04x%04x',
            mt_rand(0, 0xffff), mt_rand(0, 0xffff), mt_rand(0, 0xffff),
            mt_rand(0, 0x0fff) | 0x4000,
            mt_rand(0, 0x3fff) | 0x8000,
            mt_rand(0, 0xffff), mt_rand(0, 0xffff), mt_rand(0, 0xffff)
        );
    }

    public function getUuid(): string { return $this->uuid; }
}

// Mix in exactly what each class needs
class Article
{
    use Timestampable, SoftDeletable, HasUuid;

    public function __construct(private string $title)
    {
        $this->initTimestamps();
        $this->initUuid();
    }
}

class User
{
    use Timestampable, HasUuid; // No soft delete for users

    public function __construct(private string $email)
    {
        $this->initTimestamps();
        $this->initUuid();
    }
}

class Order
{
    use Timestampable; // Only timestamps for orders

    public function __construct(private float $total)
    {
        $this->initTimestamps();
    }
}
```

---

### Trait Conflict Resolution

```php
<?php

trait Logger
{
    public function log(string $message): void
    {
        echo "[Logger] $message\n";
    }
}

trait Auditor
{
    public function log(string $message): void  // CONFLICT — same method name
    {
        echo "[Auditor] $message\n";
    }
}

class Service
{
    use Logger, Auditor {
        Logger::log  insteadof Auditor;   // use Logger's version
        Auditor::log as auditLog;          // keep Auditor's version under alias
    }
}

$s = new Service();
$s->log("Payment processed");      // [Logger] Payment processed
$s->auditLog("Payment processed"); // [Auditor] Payment processed
```

---

### When to Use Traits vs Alternatives

```
Use Trait when:
  ✅ Same behaviour needed in unrelated classes (no common parent possible)
  ✅ Pure behaviour — no or minimal state
  ✅ Framework cross-cutting concerns (timestamps, soft delete, uuid)
  ✅ Testing helpers (MockeryTrait, DataProviderTrait)

Use Composition instead when:
  ❌ Trait accesses $this->property of the class — tight coupling
  ❌ Behaviour changes based on class context
  ❌ You need to swap implementations at runtime

Use Inheritance instead when:
  ❌ Classes genuinely share IS-A relationship
  ❌ Template Method pattern is more appropriate
```

---

## 6. Anonymous Classes

Anonymous classes — classes without a name — useful for one-off implementations.

```php
<?php

interface Logger
{
    public function log(string $message): void;
}

// Real logger
class FileLogger implements Logger
{
    public function log(string $message): void
    {
        file_put_contents('/var/log/app.log', $message . "\n", FILE_APPEND);
    }
}

// Anonymous class — for testing / one-time use
function createNullLogger(): Logger
{
    return new class implements Logger
    {
        public function log(string $message): void {} // does nothing — silent
    };
}

// Anonymous class with constructor
function createPrefixLogger(string $prefix): Logger
{
    return new class($prefix) implements Logger
    {
        public function __construct(private string $prefix) {}

        public function log(string $message): void
        {
            echo "[{$this->prefix}] $message\n";
        }
    };
}

$logger = createPrefixLogger("API");
$logger->log("Request received"); // [API] Request received

// Anonymous class extending — testing base class in isolation
abstract class BaseProcessor
{
    abstract protected function process(array $data): array;

    final public function run(array $data): array
    {
        $processed = $this->process($data);
        return array_values($processed);
    }
}

// Test the template method without a real subclass
$processor = new class extends BaseProcessor
{
    protected function process(array $data): array
    {
        return array_map('strtoupper', $data);
    }
};

print_r($processor->run(['hello', 'world'])); // ['HELLO', 'WORLD']
```

---

## 7. Object Iteration

Making objects iterable with `Iterator` and `IteratorAggregate`.

```php
<?php

// ── Iterator — implement iteration yourself ──────────────────────
class NumberRange implements \Iterator
{
    private int $current;

    public function __construct(
        private int $start,
        private int $end,
        private int $step = 1
    ) {
        $this->current = $start;
    }

    public function current(): int  { return $this->current; }
    public function key(): int      { return ($this->current - $this->start) / $this->step; }
    public function next(): void    { $this->current += $this->step; }
    public function rewind(): void  { $this->current = $this->start; }
    public function valid(): bool   { return $this->current <= $this->end; }
}

$evens = new NumberRange(2, 10, 2);
foreach ($evens as $key => $value) {
    echo "$key: $value\n"; // 0:2, 1:4, 2:6, 3:8, 4:10
}

// ── IteratorAggregate — delegate to inner iterator ─────────────
class UserCollection implements \IteratorAggregate
{
    private array $users = [];

    public function add(User $user): void { $this->users[] = $user; }

    // Delegate to ArrayIterator — simpler than implementing all 5 methods
    public function getIterator(): \ArrayIterator
    {
        return new \ArrayIterator($this->users);
    }

    public function count(): int { return count($this->users); }
}

// ── Generator-based iteration ──────────────────────────────────
class LargeFileReader
{
    public function lines(string $path): \Generator
    {
        $handle = fopen($path, 'r');
        while (($line = fgets($handle)) !== false) {
            yield trim($line);  // yields one line at a time — no memory spike
        }
        fclose($handle);
    }
}

$reader = new LargeFileReader();
foreach ($reader->lines('/tmp/huge.csv') as $i => $line) {
    echo "Line $i: $line\n";
    if ($i > 1000) break; // stop early — generator handles cleanup
}
```

---

## 8. Late Static Binding

Covered in detail in Pillar 3 — here is the practical summary.

```php
<?php

class Registry
{
    private static array $instances = [];

    public static function getInstance(): static
    {
        $class = static::class; // ← late static binding — actual called class
        if (!isset(self::$instances[$class])) {
            self::$instances[$class] = new static();
        }
        return self::$instances[$class];
    }

    public static function whoAmI(): string
    {
        return static::class; // returns ACTUAL class, not Registry
    }
}

class UserRegistry extends Registry {}
class ProductRegistry extends Registry {}

$u = UserRegistry::getInstance();    // creates UserRegistry
$p = ProductRegistry::getInstance(); // creates ProductRegistry

var_dump($u === UserRegistry::getInstance());   // true — same instance
var_dump($u === ProductRegistry::getInstance()); // false — different registry

echo UserRegistry::whoAmI();    // UserRegistry
echo ProductRegistry::whoAmI(); // ProductRegistry
echo Registry::whoAmI();        // Registry
```

---

## Quick Reference — All Additional Concepts

### SOLID at a Glance

| Principle                 | Rule                     | Violation Sign                           |
| ------------------------- | ------------------------ | ---------------------------------------- |
| **S**ingle Responsibility | One reason to change     | Class has DB + Email + Report            |
| **O**pen/Closed           | Extend, don't modify     | if/elseif grows per feature              |
| **L**iskov                | Subclass substitutable   | Child throws for parent method           |
| **I**nterface Segregation | No forced unused methods | Implements methods that throw            |
| **D**ependency Inversion  | Depend on abstractions   | `new ConcreteClass()` inside constructor |

### Magic Methods at a Glance

| Method                 | Triggered By                            |
| ---------------------- | --------------------------------------- |
| `__construct`          | `new ClassName()`                       |
| `__destruct`           | Object goes out of scope                |
| `__get($name)`         | Read undefined/inaccessible property    |
| `__set($name, $val)`   | Write undefined/inaccessible property   |
| `__isset($name)`       | `isset()` on undefined property         |
| `__unset($name)`       | `unset()` on undefined property         |
| `__toString`           | `(string) $obj` or `echo $obj`          |
| `__invoke(...)`        | `$obj(...)` — object called as function |
| `__call($name, $args)` | Undefined instance method called        |
| `__callStatic`         | Undefined static method called          |
| `__clone`              | `clone $obj`                            |
| `__serialize`          | `serialize($obj)`                       |
| `__unserialize`        | `unserialize($data)`                    |
| `__debugInfo`          | `var_dump($obj)`                        |

### Object Cloning Rules

```
clone $obj           → shallow copy (scalar values copied, objects shared)
__clone()            → hook to deep-copy nested objects
with() pattern       → immutable objects return modified clone
```

### Object Comparison Rules

```
$a == $b    → same class + same property values → true
$a === $b   → exactly the same instance in memory → true
Always use explicit equals() method for domain equality
Use === only for identity checks
```

### Trait Rules

```
✅ Use for: cross-cutting concerns, testing helpers, stateless behaviour
❌ Avoid: accessing $this->property, complex state, conflict-prone APIs
Conflict: use `insteadof` to resolve, `as` to alias
```

---

_← [Pillar 4 — Polymorphism](./04_polymorphism.md)_

---

_Complete OOP Series:_
_[01 Encapsulation](./01_encapsulation.md) · [02 Abstraction](./02_abstraction.md) · [03 Inheritance](./03_inheritance.md) · [04 Polymorphism](./04_polymorphism.md) · [05 Additional](./05_additional.md)_
