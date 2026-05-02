# Pillar 3: Inheritance

### IS-A Relationships · Code Reuse · Overriding · LSP · Template Method

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

> **Inheritance** = A class **IS A** more specific version of another class — getting all its properties and methods, and adding or overriding what it needs.

The test: replace "extends" with "IS A" and read it aloud.

- `Dog extends Animal` → "Dog IS AN Animal" ✅ — makes sense
- `OrderService extends Logger` → "OrderService IS A Logger" ❌ — nonsense → wrong design

---

## 2. For a 5-Year-Old 🧸

### The Family Tree Story 👨‍👩‍👧

Think about your **family**.

You got some things from your parents automatically — their eye colour, some personality traits.
You didn't have to "build" yourself from scratch.

```
Grandparent (has: two eyes, two hands, can breathe, can walk)
    │
    ├── Parent (inherits: eyes, hands, breathing, walking)
    │         (adds: reads books, drives a car)
    │
    └── You (inherits: eyes, hands, breathing, walking, reads, drives)
             (adds: plays video games, uses phone)
```

**You inherited** the basic stuff — and added your own things on top.

In programming:

```
Animal (can: eat, sleep, breathe)
  │
  ├── Dog (inherits: eat, sleep, breathe)
  │       (adds: fetch, wag tail)
  │       (changes: makeSound → "Woof!" instead of generic sound)
  │
  └── Cat (inherits: eat, sleep, breathe)
          (adds: purr, climb)
          (changes: makeSound → "Meow!" instead of generic sound)
```

Dog and Cat didn't have to write `eat()` and `sleep()` themselves.
They **inherited** those from Animal — and just added their own special things.

---

## 3. For a Mid-Level Developer 💻

### WITHOUT Inheritance — The Problem ❌

```php
<?php

// No inheritance — pure duplication
class Dog
{
    private string $name;
    private int    $age;
    private float  $weight;

    public function __construct(string $name, int $age, float $weight)
    {
        $this->name   = $name;
        $this->age    = $age;
        $this->weight = $weight;
    }

    // Duplicated in EVERY animal class ↓
    public function eat(string $food): string  { return "{$this->name} eats $food"; }
    public function sleep(int $hours): string  { return "{$this->name} sleeps $hours hours"; }
    public function breathe(): string          { return "{$this->name} breathes"; }
    public function getName(): string          { return $this->name; }
    public function getAge(): int              { return $this->age; }
    // Duplicated in EVERY animal class ↑

    public function makeSound(): string { return "{$this->name}: Woof!"; }
    public function fetch(): string     { return "{$this->name} fetches the ball!"; }
}

class Cat
{
    private string $name;
    private int    $age;
    private float  $weight;

    public function __construct(string $name, int $age, float $weight)
    {
        $this->name   = $name;
        $this->age    = $age;
        $this->weight = $weight;
    }

    // SAME CODE AGAIN — exact copy ↓
    public function eat(string $food): string  { return "{$this->name} eats $food"; }
    public function sleep(int $hours): string  { return "{$this->name} sleeps $hours hours"; }
    public function breathe(): string          { return "{$this->name} breathes"; }
    public function getName(): string          { return $this->name; }
    public function getAge(): int              { return $this->age; }
    // SAME CODE AGAIN ↑

    public function makeSound(): string { return "{$this->name}: Meow!"; }
    public function purr(): string      { return "{$this->name} purrs..."; }
}

// Cannot treat Dog and Cat as the same type:
$animals = [new Dog("Rex", 3, 25.0), new Cat("Kitty", 5, 4.5)];
foreach ($animals as $animal) {
    // echo $animal->eat("fish"); ← ❌ PHP doesn't know they both have eat()
}
```

---

### WITH Inheritance — The Solution ✅

```php
<?php

abstract class Animal
{
    public function __construct(
        protected string $name,
        protected int    $age,
        protected float  $weight
    ) {}

    // Shared — implemented ONCE
    public function eat(string $food): string  { return "{$this->name} eats $food"; }
    public function sleep(int $hours): string  { return "{$this->name} sleeps $hours hours"; }
    public function breathe(): string          { return "{$this->name} breathes"; }
    public function getName(): string          { return $this->name; }
    public function getAge(): int              { return $this->age; }

    // Abstract — MUST be different per animal
    abstract public function makeSound(): string;

    // Concrete with sensible default — CAN be overridden
    public function describe(): string
    {
        return "{$this->name} is a " . static::class . ", age {$this->age}";
    }
}

class Dog extends Animal
{
    public function makeSound(): string { return "{$this->name}: Woof!"; }
    public function fetch(): string     { return "{$this->name} fetches the ball!"; }
}

class Cat extends Animal
{
    public function makeSound(): string { return "{$this->name}: Meow!"; }
    public function purr(): string      { return "{$this->name} purrs..."; }

    // Override describe() — add cat-specific info
    public function describe(): string
    {
        return parent::describe() . " (indoor cat)";
    }
}

class Bird extends Animal
{
    private bool $canFly;

    public function __construct(string $name, int $age, float $weight, bool $canFly = true)
    {
        parent::__construct($name, $age, $weight); // call parent constructor
        $this->canFly = $canFly;
    }

    public function makeSound(): string { return "{$this->name}: Tweet!"; }

    public function fly(): string
    {
        return $this->canFly
            ? "{$this->name} soars through the sky!"
            : "{$this->name} cannot fly (flightless bird)";
    }
}

// Polymorphism — treat all as Animal
$animals = [
    new Dog("Rex", 3, 25.0),
    new Cat("Kitty", 5, 4.5),
    new Bird("Tweety", 2, 0.5),
    new Bird("Pingu", 4, 5.0, false),
];

foreach ($animals as $animal) {
    echo $animal->eat("fish") . "\n";   // shared from Animal
    echo $animal->makeSound() . "\n";   // each animal's own version
    echo $animal->describe() . "\n";    // default or overridden
    echo "---\n";
}
```

---

### Method Overriding

```php
<?php

class Vehicle
{
    public function __construct(protected string $brand) {}

    public function start(): string
    {
        return "{$this->brand}: Starting engine...";
    }

    public function describe(): string
    {
        return "Vehicle: {$this->brand}";
    }
}

class ElectricCar extends Vehicle
{
    public function __construct(string $brand, private int $batteryPercent)
    {
        parent::__construct($brand);
    }

    // Override — completely replace parent behaviour
    public function start(): string
    {
        return "{$this->brand}: Silently powering up... 🔋 {$this->batteryPercent}%";
    }

    // Override — extend parent behaviour with parent::
    public function describe(): string
    {
        return parent::describe() . " (Electric, Battery: {$this->batteryPercent}%)";
    }
}

class HybridCar extends Vehicle
{
    public function start(): string
    {
        // Use electric mode first, then parent's engine start
        return "Hybrid mode: " . parent::start() . " (+ electric assist)";
    }
}

$car     = new Vehicle("Toyota");
$tesla   = new ElectricCar("Tesla", 87);
$prius   = new HybridCar("Prius");

echo $car->start()   . "\n"; // Toyota: Starting engine...
echo $tesla->start() . "\n"; // Tesla: Silently powering up... 🔋 87%
echo $prius->start() . "\n"; // Hybrid mode: Prius: Starting engine... (+ electric assist)

echo $tesla->describe() . "\n"; // Vehicle: Tesla (Electric, Battery: 87%)
```

---

### Constructor Chaining

```php
<?php

class Person
{
    public function __construct(
        protected string $name,
        protected string $email
    ) {}
}

class Employee extends Person
{
    public function __construct(
        string          $name,
        string          $email,
        private string  $department,
        private float   $salary
    ) {
        parent::__construct($name, $email); // MUST call parent constructor
    }

    public function getInfo(): string
    {
        return "{$this->name} ({$this->email}) — {$this->department} — €{$this->salary}/yr";
    }
}

class Manager extends Employee
{
    private array $reports = [];

    public function __construct(
        string $name,
        string $email,
        string $department,
        float  $salary,
        private int $teamSize
    ) {
        parent::__construct($name, $email, $department, $salary);
    }

    public function addReport(Employee $emp): void
    {
        $this->reports[] = $emp;
    }

    public function getInfo(): string
    {
        return parent::getInfo() . " — manages {$this->teamSize} people";
    }
}

$emp     = new Employee("Ali", "ali@company.com", "Engineering", 75000);
$manager = new Manager("Shivani", "shivani@company.com", "Engineering", 95000, 6);

echo $emp->getInfo()     . "\n";
echo $manager->getInfo() . "\n";
```

---

### Preventing Inheritance — `final`

```php
<?php

// final class — nobody can extend this
final class Uuid
{
    private string $value;

    private function __construct(string $value)
    {
        if (!preg_match('/^[0-9a-f]{8}-[0-9a-f]{4}-4[0-9a-f]{3}-[89ab][0-9a-f]{3}-[0-9a-f]{12}$/i', $value)) {
            throw new \InvalidArgumentException("Invalid UUID: $value");
        }
        $this->value = $value;
    }

    public static function generate(): self
    {
        return new self(sprintf(
            '%04x%04x-%04x-%04x-%04x-%04x%04x%04x',
            mt_rand(0, 0xffff), mt_rand(0, 0xffff),
            mt_rand(0, 0xffff),
            mt_rand(0, 0x0fff) | 0x4000,
            mt_rand(0, 0x3fff) | 0x8000,
            mt_rand(0, 0xffff), mt_rand(0, 0xffff), mt_rand(0, 0xffff)
        ));
    }

    public static function fromString(string $value): self { return new self($value); }

    public function __toString(): string { return $this->value; }
    public function equals(self $other): bool { return $this->value === $other->value; }
}

// class SpecialUuid extends Uuid {} ← ❌ Fatal error: Class marked as final

// final METHOD — class is inheritable but this method is not overridable
class BaseController
{
    final protected function json(array $data, int $status = 200): string
    {
        http_response_code($status);
        header('Content-Type: application/json');
        return json_encode($data);
    }
}

class UserController extends BaseController
{
    public function index(): string
    {
        return $this->json(['users' => []]);
    }

    // Cannot override json() — it's final
}
```

---

## 4. For an Expert Developer 🧠

### Liskov Substitution Principle (LSP)

LSP is the rule that makes inheritance correct or wrong.

> Any subclass must be **fully substitutable** for its parent class without breaking the program's correctness.

```php
<?php

// ❌ Classic LSP violation: Rectangle / Square
class Rectangle
{
    public function __construct(
        protected int $width,
        protected int $height
    ) {}

    public function setWidth(int $w): void  { $this->width  = $w; }
    public function setHeight(int $h): void { $this->height = $h; }
    public function area(): int             { return $this->width * $this->height; }
}

class Square extends Rectangle
{
    public function setWidth(int $w): void  { $this->width = $this->height = $w; }
    public function setHeight(int $h): void { $this->width = $this->height = $h; }
}

// Code that works correctly with Rectangle:
function assertArea(Rectangle $r): void
{
    $r->setWidth(4);
    $r->setHeight(5);
    assert($r->area() === 20, "Expected area 20, got " . $r->area());
}

assertArea(new Rectangle(0, 0)); // ✅ area = 20
assertArea(new Square(0));       // ❌ area = 25 (setHeight overwrote width!)

// Square IS-A Rectangle mathematically — NOT behaviourally
// The IS-A test fails behaviourally → inheritance is WRONG

// ✅ Fix: separate hierarchy
interface Shape { public function area(): int; }

final class Rectangle implements Shape
{
    public function __construct(private int $width, private int $height) {}
    public function area(): int { return $this->width * $this->height; }
}

final class Square implements Shape
{
    public function __construct(private int $side) {}
    public function area(): int { return $this->side ** 2; }
}

// Both substitutable for Shape — LSP holds
```

---

### Covariant Return Types (PHP 7.4+)

```php
<?php

class Animal
{
    public function create(): static { return new static(); }

    // Covariant: child can return a MORE SPECIFIC type
    public function clone(): static { return clone $this; }
}

class Dog extends Animal
{
    private string $breed = 'unknown';

    public function setBreed(string $breed): static
    {
        $this->breed = $breed;
        return $this;
    }

    public function getBreed(): string { return $this->breed; }
}

$dog      = new Dog();
$cloned   = $dog->clone();   // returns Dog, not Animal — covariance
$specific = $dog->setBreed('Labrador'); // fluent — returns Dog

echo $cloned::class;          // Dog ← correct type preserved
echo $specific->getBreed();   // Labrador
```

---

### Abstract Class — Template Method Pattern (Deep)

```php
<?php

abstract class DataMigration
{
    private array $log = [];

    // Template method — SEALED algorithm — children fill in steps
    final public function run(): MigrationResult
    {
        $this->log('Starting migration: ' . static::class);

        $this->beforeMigration();

        try {
            $source      = $this->readSource();
            $this->log("Read " . count($source) . " records");

            $transformed = $this->transform($source);
            $this->log("Transformed " . count($transformed) . " records");

            $validated   = $this->validate($transformed);
            $this->log("Validated: " . count($validated) . " passed");

            $saved       = $this->persist($validated);
            $this->log("Saved " . count($saved) . " records");

            $this->afterSuccess($saved);

            return MigrationResult::success($saved, $this->log);
        } catch (\Throwable $e) {
            $this->log("FAILED: " . $e->getMessage());
            $this->afterFailure($e);
            return MigrationResult::failure($e, $this->log);
        }
    }

    // Steps that MUST be implemented
    abstract protected function readSource(): array;
    abstract protected function transform(array $source): array;
    abstract protected function persist(array $data): array;

    // Steps with defaults — CAN be overridden
    protected function validate(array $data): array
    {
        return array_filter($data, fn($row) => !empty($row));
    }

    // Hooks — optional, do nothing by default
    protected function beforeMigration(): void {}
    protected function afterSuccess(array $saved): void {}
    protected function afterFailure(\Throwable $e): void {}

    private function log(string $message): void
    {
        $this->log[] = date('H:i:s') . " — $message";
    }
}

class UserCsvMigration extends DataMigration
{
    public function __construct(private string $csvPath) {}

    protected function readSource(): array
    {
        $lines = file($this->csvPath, FILE_IGNORE_NEW_LINES | FILE_SKIP_EMPTY_LINES);
        $headers = str_getcsv(array_shift($lines));
        return array_map(fn($line) => array_combine($headers, str_getcsv($line)), $lines);
    }

    protected function transform(array $source): array
    {
        return array_map(fn($row) => [
            'name'       => trim($row['name']),
            'email'      => strtolower(trim($row['email'])),
            'created_at' => date('Y-m-d H:i:s'),
        ], $source);
    }

    protected function validate(array $data): array
    {
        return array_filter($data, fn($row) =>
            filter_var($row['email'], FILTER_VALIDATE_EMAIL)
            && strlen($row['name']) > 0
        );
    }

    protected function persist(array $data): array
    {
        // INSERT INTO users ...
        return $data;
    }

    protected function beforeMigration(): void
    {
        echo "Backing up users table...\n";
    }

    protected function afterFailure(\Throwable $e): void
    {
        echo "Sending failure alert to Slack...\n";
    }
}
```

---

### Late Static Binding — `static` vs `self`

```php
<?php

class Base
{
    // self:: always refers to Base — even when called on Child
    public static function createWithSelf(): self
    {
        return new self(); // ALWAYS creates Base — not the calling class
    }

    // static:: refers to the ACTUAL called class — late binding
    public static function createWithStatic(): static
    {
        return new static(); // creates whatever class was actually called
    }

    public static function whoAmI(): string
    {
        return "self: " . self::class . " | static: " . static::class;
    }
}

class Child extends Base {}

$b1 = Base::createWithSelf();    // Base ✅
$b2 = Child::createWithSelf();   // Base ❌ — wrong! should be Child
$b3 = Child::createWithStatic(); // Child ✅ — correct

echo Base::whoAmI();   // self: Base | static: Base
echo Child::whoAmI();  // self: Base | static: Child ← late static binding

// Practical: Builder pattern with inheritance
class QueryBuilder
{
    protected array $wheres = [];
    protected array $orders = [];

    public function where(string $condition): static
    {
        $clone          = clone $this;
        $clone->wheres[] = $condition;
        return $clone; // returns ACTUAL type — Child returns Child, not QueryBuilder
    }

    public function orderBy(string $column): static
    {
        $clone           = clone $this;
        $clone->orders[]  = $column;
        return $clone;
    }
}

class UserQueryBuilder extends QueryBuilder
{
    private ?int $minAge = null;

    public function olderThan(int $age): static
    {
        $clone          = clone $this;
        $clone->minAge  = $age;
        $clone->wheres[] = "age > $age";
        return $clone;
    }
}

// Method chaining works correctly — returns UserQueryBuilder, not QueryBuilder
$query = (new UserQueryBuilder())
    ->where("active = 1")    // returns UserQueryBuilder
    ->olderThan(18)           // ✅ available — type preserved
    ->orderBy("name");        // returns UserQueryBuilder
```

---

### Multi-Level Inheritance — When It Gets Deep

```php
<?php

// Symfony-style — controlled deep hierarchy (acceptable because framework-defined)
class AbstractController         // Level 1 — framework base
{
    protected function json(array $data): string { return json_encode($data); }
    protected function redirect(string $url): void { header("Location: $url"); }
}

class AuthenticatedController extends AbstractController  // Level 2 — app base
{
    protected function requireAuth(): void
    {
        // Check session/JWT — all auth controllers need this
    }
    protected function getCurrentUser(): ?User { return null; }
}

class AdminController extends AuthenticatedController    // Level 3 — role check
{
    protected function requireAdmin(): void
    {
        $this->requireAuth();
        // check admin role
    }
}

class UserManagementController extends AdminController   // Level 4 — feature
{
    public function listUsers(): string
    {
        $this->requireAdmin();
        return $this->json(['users' => []]);
    }
}

// 4 levels is approaching the limit of acceptable depth
// Rule: >3 levels → reconsider the design
```

---

## 5. For the Critics 🔥

### Criticism 1 — Inheritance Is the Most Misused OOP Feature

**The Argument:**

> "90% of `extends` in PHP codebases are wrong. Either it's reusing code (should be composition or trait), or it's sharing a type (should be interface), or it's a framework base class nobody reads. Developers reach for inheritance because it's the first OOP tool they learn — not because it's right."

**Largely valid.**
The "IS-A" test catches most misuses but developers don't apply it. `UserService extends BaseService` — is UserService a BaseService? No. It just needs some shared helper methods. That's a composition or trait problem masquerading as inheritance. The rule: if the parent can be replaced with an interface + injected collaborator — do that instead.

---

### Criticism 2 — Deep Hierarchies Are Impossible to Maintain

**The Argument:**

> "I have worked on codebases with 6-7 level inheritance hierarchies. To understand any method you need to trace through all ancestor classes. Overrides call `parent::` which calls another `parent::` — it's a call stack archaeology project. The feature that seems to add code reuse removes readability entirely."

**Correct — and preventable.**
The rule of thumb: no more than 2-3 levels of inheritance in application code. Framework hierarchies (Symfony, Laravel) can go deeper because they're authored by experts who carefully manage the hierarchy. Application code rarely justifies it. If you have 4+ levels — the design has drifted. Flatten via composition.

---

### Criticism 3 — `parent::` Calls Are Hidden Coupling

**The Argument:**

> "`parent::__construct()`, `parent::method()` — every time you call parent, you're coupling yourself to the parent's implementation. If the parent constructor changes signature, every child breaks. This is tight coupling with extra steps."

**True — this is exactly why constructor promotion and value objects exist.**
PHP 8.0's constructor promotion reduces one source of parent coupling. The deeper issue: calling `parent::method()` means the child knows about the parent's implementation — not just its interface. This is inheritance's fundamental coupling cost. Mitigated by: not extending what you don't own, keeping hierarchies shallow, and using `final` on methods that shouldn't be overridden.

---

## 6. Quick Reference

```
Inheritance Rules:
  ✅ Use when IS-A is genuinely true and stable
  ✅ Maximum 2-3 levels in application code
  ✅ Always call parent::__construct() if parent has one
  ✅ Use `final` to seal the hierarchy at the right point
  ✅ Use `abstract` to force children to implement key steps
  ✅ Respect LSP — child must fully substitute parent
  ❌ Don't extend just to reuse methods (use trait/composition)
  ❌ Don't extend classes you don't own (use composition)
  ❌ Don't override to throw exception (LSP violation)
  ❌ Don't check instanceof in the parent (Demeter violation)
```

| Keyword             | Meaning                                         |
| ------------------- | ----------------------------------------------- |
| `extends`           | This class IS-A more specific version of parent |
| `abstract class`    | Cannot be instantiated — must be extended       |
| `abstract function` | Child MUST implement this                       |
| `final class`       | Cannot be extended                              |
| `final function`    | Cannot be overridden                            |
| `parent::`          | Call parent's version of method                 |
| `static::`          | Late static binding — actual called class       |
| `self::`            | Always THIS class — no late binding             |

---

_← [Pillar 2 — Abstraction](./02_abstraction.md) · [Pillar 4 — Polymorphism →](./04_polymorphism.md)_
