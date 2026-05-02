# Pillar 4: Polymorphism

### One Interface · Many Forms · Runtime Dispatch · Duck Typing

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

> **Polymorphism** = One interface, many implementations — the same method call behaves differently depending on which object receives it.

From Greek: **poly** (many) + **morphe** (form) = many forms.

Three types in PHP:

- **Subtype Polymorphism** — via inheritance / interfaces (most common)
- **Parametric Polymorphism** — via generics/templates (PHPDoc in PHP)
- **Ad-hoc Polymorphism** — via method overloading (limited in PHP)

---

## 2. For a 5-Year-Old 🧸

### The "Make Sound" Story 🎵

Imagine you are a zookeeper. Every morning you walk through the zoo and say to each animal:

**"Make your sound!"**

```
You say: "Make your sound!"

Dog hears it  → "Woof! Woof!"
Cat hears it  → "Meooow!"
Duck hears it → "Quack! Quack!"
Cow hears it  → "Mooooo!"
Lion hears it → "ROAAAAAR!"
```

You said the **exact same thing** to each animal.
But each animal responded in its **own way**.

THAT is polymorphism.

**One command → Many different behaviours.**

You don't need to say "Dog, bark!" and "Cat, meow!" separately.
You just say "Make your sound" — and each one knows what to do.

In code:

```php
foreach ($animals as $animal) {
    echo $animal->makeSound(); // Same call — different result every time
}
```

---

## 3. For a Mid-Level Developer 💻

### WITHOUT Polymorphism — The Problem ❌

```php
<?php

// Without polymorphism — giant if/elseif chains everywhere
class Animal
{
    public string $type;
    public string $name;

    public function __construct(string $type, string $name)
    {
        $this->type = $type;
        $this->name = $name;
    }
}

// Every function that uses Animal must know about ALL types
function makeAnimalSound(Animal $animal): string
{
    if ($animal->type === 'dog') {
        return "{$animal->name}: Woof!";
    } elseif ($animal->type === 'cat') {
        return "{$animal->name}: Meow!";
    } elseif ($animal->type === 'duck') {
        return "{$animal->name}: Quack!";
    } elseif ($animal->type === 'cow') {
        return "{$animal->name}: Moo!";
    }
    // Add lion? Edit this function. Add snake? Edit this function.
    // This if/elseif grows FOREVER
    return "Unknown animal sound";
}

function describeMovement(Animal $animal): string
{
    // SAME if/elseif AGAIN — duplicated across every function
    if ($animal->type === 'dog') return "{$animal->name} runs";
    elseif ($animal->type === 'duck') return "{$animal->name} waddles and flies";
    elseif ($animal->type === 'fish') return "{$animal->name} swims";
    return "moves somehow";
}

function calculateFoodCost(Animal $animal, int $days): float
{
    // SAME if/elseif YET AGAIN
    if ($animal->type === 'dog')  return $days * 2.50;
    elseif ($animal->type === 'cat')  return $days * 1.80;
    elseif ($animal->type === 'lion') return $days * 15.00;
    return $days * 1.0;
}

$dog  = new Animal('dog', 'Rex');
$cat  = new Animal('cat', 'Kitty');
$duck = new Animal('duck', 'Donald');

// Adding a new animal type = change EVERY function above
// Open/Closed Principle violated every time
```

---

### WITH Polymorphism — The Solution ✅

```php
<?php

abstract class Animal
{
    public function __construct(protected string $name) {}

    // Each animal defines its OWN behaviour
    abstract public function makeSound(): string;
    abstract public function move(): string;
    abstract public function getFoodCostPerDay(): float;

    // Shared — same for all
    public function getName(): string { return $this->name; }

    public function getFoodCost(int $days): float
    {
        return $this->getFoodCostPerDay() * $days;
    }
}

class Dog extends Animal
{
    public function makeSound(): string          { return "{$this->name}: Woof! 🐕"; }
    public function move(): string               { return "{$this->name} runs"; }
    public function getFoodCostPerDay(): float   { return 2.50; }
}

class Cat extends Animal
{
    public function makeSound(): string          { return "{$this->name}: Meow! 🐈"; }
    public function move(): string               { return "{$this->name} sneaks"; }
    public function getFoodCostPerDay(): float   { return 1.80; }
}

class Duck extends Animal
{
    public function makeSound(): string          { return "{$this->name}: Quack! 🦆"; }
    public function move(): string               { return "{$this->name} waddles and flies"; }
    public function getFoodCostPerDay(): float   { return 0.75; }
}

class Lion extends Animal
{
    public function makeSound(): string          { return "{$this->name}: ROAAAAR! 🦁"; }
    public function move(): string               { return "{$this->name} prowls"; }
    public function getFoodCostPerDay(): float   { return 15.00; }
}

// Adding Snake: 1 new class, ZERO changes to existing code
class Snake extends Animal
{
    public function makeSound(): string          { return "{$this->name}: Hissss 🐍"; }
    public function move(): string               { return "{$this->name} slithers"; }
    public function getFoodCostPerDay(): float   { return 0.50; }
}

// These functions NEVER change regardless of how many animals you add
function morningRoutine(array $animals): void
{
    foreach ($animals as $animal) {
        echo $animal->makeSound() . "\n"; // Polymorphic call
        echo $animal->move()      . "\n";
    }
}

function monthlyFoodBudget(array $animals): float
{
    return array_sum(array_map(
        fn(Animal $a) => $a->getFoodCost(30), // Polymorphic — each animal knows its cost
        $animals
    ));
}

$zoo = [new Dog("Rex"), new Cat("Kitty"), new Duck("Donald"), new Lion("Simba"), new Snake("Sss")];

morningRoutine($zoo);
echo "Monthly food cost: €" . number_format(monthlyFoodBudget($zoo), 2) . "\n";
// Adding new animal: create class, add to $zoo array — nothing else changes
```

---

### Interface-Based Polymorphism

```php
<?php

// Polymorphism doesn't require inheritance — interfaces work too
// And are MORE flexible (unrelated classes can share an interface)

interface Drawable
{
    public function draw(): string;
    public function getArea(): float;
    public function getPerimeter(): float;
}

class Circle implements Drawable
{
    public function __construct(private float $radius) {}

    public function draw(): string        { return "Drawing circle ⭕ r={$this->radius}"; }
    public function getArea(): float      { return M_PI * $this->radius ** 2; }
    public function getPerimeter(): float { return 2 * M_PI * $this->radius; }
}

class Rectangle implements Drawable
{
    public function __construct(private float $w, private float $h) {}

    public function draw(): string        { return "Drawing rectangle 🔲 {$this->w}×{$this->h}"; }
    public function getArea(): float      { return $this->w * $this->h; }
    public function getPerimeter(): float { return 2 * ($this->w + $this->h); }
}

class Triangle implements Drawable
{
    public function __construct(
        private float $a,
        private float $b,
        private float $c,
        private float $height
    ) {}

    public function draw(): string        { return "Drawing triangle 🔺 sides {$this->a},{$this->b},{$this->c}"; }
    public function getArea(): float      { return 0.5 * $this->a * $this->height; }
    public function getPerimeter(): float { return $this->a + $this->b + $this->c; }
}

// Canvas works with ANY Drawable — no knowledge of specific shapes
class Canvas
{
    /** @var Drawable[] */
    private array $shapes = [];

    public function add(Drawable $shape): void
    {
        $this->shapes[] = $shape;
    }

    public function render(): void
    {
        foreach ($this->shapes as $shape) {
            echo $shape->draw() . "\n";
            echo "  Area: " . number_format($shape->getArea(), 2) . "\n";
            echo "  Perimeter: " . number_format($shape->getPerimeter(), 2) . "\n";
        }
    }

    public function totalArea(): float
    {
        return array_sum(array_map(fn(Drawable $s) => $s->getArea(), $this->shapes));
    }
}

$canvas = new Canvas();
$canvas->add(new Circle(5.0));
$canvas->add(new Rectangle(4.0, 6.0));
$canvas->add(new Triangle(3.0, 4.0, 5.0, 4.0));

$canvas->render();
echo "Total area: " . number_format($canvas->totalArea(), 2) . "\n";
```

---

### Real-World Polymorphism — Notification System

```php
<?php

interface Notification
{
    public function send(string $to, string $subject, string $body): bool;
    public function getChannelName(): string;
}

class EmailNotification implements Notification
{
    public function __construct(private string $smtpHost) {}

    public function send(string $to, string $subject, string $body): bool
    {
        echo "📧 Email → {$to}: [{$subject}] {$body}\n";
        return true; // would use SMTP in reality
    }

    public function getChannelName(): string { return 'email'; }
}

class SlackNotification implements Notification
{
    public function __construct(private string $webhookUrl) {}

    public function send(string $to, string $subject, string $body): bool
    {
        echo "💬 Slack → #{$to}: *{$subject}* {$body}\n";
        return true; // would use Slack API in reality
    }

    public function getChannelName(): string { return 'slack'; }
}

class SmsNotification implements Notification
{
    public function __construct(private string $twilioSid) {}

    public function send(string $to, string $subject, string $body): bool
    {
        echo "📱 SMS → {$to}: {$subject}: {$body}\n";
        return true;
    }

    public function getChannelName(): string { return 'sms'; }
}

class PushNotification implements Notification
{
    public function send(string $to, string $subject, string $body): bool
    {
        echo "🔔 Push → {$to}: {$subject}\n";
        return true;
    }

    public function getChannelName(): string { return 'push'; }
}

// NotificationDispatcher uses polymorphism — never knows WHICH channel
class NotificationDispatcher
{
    /** @var Notification[] */
    private array $channels = [];

    public function addChannel(Notification $channel): void
    {
        $this->channels[$channel->getChannelName()] = $channel;
    }

    public function dispatch(
        string $to,
        string $subject,
        string $body,
        array  $via = []             // empty = all channels
    ): array {
        $targets  = empty($via) ? $this->channels : array_intersect_key(
            $this->channels,
            array_flip($via)
        );
        $results  = [];

        foreach ($targets as $name => $channel) {
            $results[$name] = $channel->send($to, $subject, $body);
        }

        return $results;
    }
}

$dispatcher = new NotificationDispatcher();
$dispatcher->addChannel(new EmailNotification('smtp.gmail.com'));
$dispatcher->addChannel(new SlackNotification('https://hooks.slack.com/xxx'));
$dispatcher->addChannel(new SmsNotification('AC_xxx'));
$dispatcher->addChannel(new PushNotification());

// Send via all channels
$dispatcher->dispatch("shivani@gmail.com", "Order Shipped", "Your order is on its way!");

// Send via specific channels only
$dispatcher->dispatch("shivani@gmail.com", "Flash Sale!", "50% off for 2 hours!", ['email', 'push']);
```

---

## 4. For an Expert Developer 🧠

### Method Overloading via `__call` and `__callStatic`

PHP doesn't have true method overloading (same name, different signatures).
`__call` and `__callStatic` provide dynamic dispatch instead.

```php
<?php

class FluentQueryBuilder
{
    private array $conditions = [];
    private array $columns    = ['*'];
    private ?int  $limitVal   = null;
    private array $orderByClauses = [];

    // Intercepts calls to undefined methods — dynamic polymorphism
    public function __call(string $name, array $arguments): static
    {
        // whereXxx($value) pattern — e.g. whereStatus('active'), whereUserId(42)
        if (str_starts_with($name, 'where')) {
            $column = strtolower(preg_replace('/([A-Z])/', '_$1', lcfirst(substr($name, 5))));
            $this->conditions[] = "$column = ?";
            // store value...
            return $this;
        }

        // orderByXxx() pattern
        if (str_starts_with($name, 'orderBy')) {
            $column = strtolower(preg_replace('/([A-Z])/', '_$1', lcfirst(substr($name, 7))));
            $this->orderByClauses[] = $column;
            return $this;
        }

        throw new \BadMethodCallException("Method not found: $name");
    }

    public static function __callStatic(string $name, array $arguments): static
    {
        $instance = new static();
        return $instance->$name(...$arguments);
    }

    public function select(string ...$columns): static
    {
        $this->columns = $columns;
        return $this;
    }

    public function limit(int $n): static
    {
        $this->limitVal = $n;
        return $this;
    }

    public function toSql(): string
    {
        $sql = "SELECT " . implode(', ', $this->columns) . " FROM table";
        if ($this->conditions) $sql .= " WHERE " . implode(' AND ', $this->conditions);
        if ($this->orderByClauses) $sql .= " ORDER BY " . implode(', ', $this->orderByClauses);
        if ($this->limitVal) $sql .= " LIMIT {$this->limitVal}";
        return $sql;
    }
}

$query = (new FluentQueryBuilder())
    ->select('id', 'name', 'email')
    ->whereStatus('active')         // __call intercepts this
    ->whereUserId(42)               // __call intercepts this
    ->orderByCreatedAt()            // __call intercepts this
    ->limit(10);

echo $query->toSql();
// SELECT id, name, email FROM table WHERE status = ? AND user_id = ? ORDER BY created_at LIMIT 10
```

---

### Covariance and Contravariance

```php
<?php

// Covariance — return type can be MORE specific in child class
class Animal {}
class Dog extends Animal {}

class AnimalFactory
{
    public function create(): Animal    // returns Animal
    {
        return new Animal();
    }
}

class DogFactory extends AnimalFactory
{
    public function create(): Dog   // ✅ Covariant — Dog IS-A Animal, more specific
    {
        return new Dog();
    }
}

// Contravariance — parameter type can be LESS specific in child class
class AnimalHandler
{
    public function handle(Dog $dog): void    // accepts Dog
    {
        echo "Handling dog\n";
    }
}

class GenericAnimalHandler extends AnimalHandler
{
    public function handle(Animal $animal): void  // ✅ Contravariant — accepts ANY Animal
    {
        echo "Handling animal\n";
    }
}

// These rules ensure LSP holds for type-safe polymorphism
// PHP 7.4+ enforces these rules
```

---

### Polymorphism + Strategy Pattern — Runtime Behaviour Swap

```php
<?php

// Strategy pattern IS polymorphism applied to algorithms

interface SortStrategy
{
    public function sort(array &$data): void;
    public function getName(): string;
}

class BubbleSortStrategy implements SortStrategy
{
    public function sort(array &$data): void
    {
        $n = count($data);
        for ($i = 0; $i < $n - 1; $i++)
            for ($j = 0; $j < $n - $i - 1; $j++)
                if ($data[$j] > $data[$j + 1])
                    [$data[$j], $data[$j + 1]] = [$data[$j + 1], $data[$j]];
    }
    public function getName(): string { return 'Bubble Sort O(n²)'; }
}

class QuickSortStrategy implements SortStrategy
{
    public function sort(array &$data): void { sort($data); }
    public function getName(): string        { return 'Quick Sort O(n log n)'; }
}

class RadixSortStrategy implements SortStrategy
{
    public function sort(array &$data): void { sort($data); } // simplified
    public function getName(): string        { return 'Radix Sort O(nk)'; }
}

class SmartSorter
{
    private SortStrategy $strategy;

    public function __construct(array $data)
    {
        // Choose strategy based on data — polymorphic dispatch at runtime
        $this->strategy = $this->selectOptimalStrategy($data);
    }

    private function selectOptimalStrategy(array $data): SortStrategy
    {
        $count = count($data);

        if ($count <= 10) return new BubbleSortStrategy();     // small data
        if ($count <= 100000) return new QuickSortStrategy();  // medium data
        return new RadixSortStrategy();                          // large data
    }

    public function sort(array &$data): void
    {
        echo "Using: {$this->strategy->getName()}\n";
        $this->strategy->sort($data);
    }

    // Swap at runtime — polymorphism in action
    public function setStrategy(SortStrategy $strategy): void
    {
        $this->strategy = $strategy;
    }
}

$data = [5, 2, 8, 1, 9, 3, 7, 4, 6];
$sorter = new SmartSorter($data);
$sorter->sort($data);
```

---

### Polymorphism + Visitor Pattern — Double Dispatch

```php
<?php

// Visitor solves: adding operations to a class hierarchy WITHOUT modifying the hierarchy
// Uses double dispatch — two polymorphic calls

interface ShapeVisitor
{
    public function visitCircle(Circle $circle): mixed;
    public function visitRectangle(Rectangle $rectangle): mixed;
    public function visitTriangle(Triangle $triangle): mixed;
}

interface Shape
{
    public function accept(ShapeVisitor $visitor): mixed;
}

class Circle implements Shape
{
    public function __construct(public readonly float $radius) {}

    public function accept(ShapeVisitor $visitor): mixed
    {
        return $visitor->visitCircle($this);  // 1st dispatch: circle tells visitor "I'm a circle"
    }
}

class Rectangle implements Shape
{
    public function __construct(
        public readonly float $width,
        public readonly float $height
    ) {}

    public function accept(ShapeVisitor $visitor): mixed
    {
        return $visitor->visitRectangle($this);
    }
}

class Triangle implements Shape
{
    public function __construct(
        public readonly float $base,
        public readonly float $height
    ) {}

    public function accept(ShapeVisitor $visitor): mixed
    {
        return $visitor->visitTriangle($this);
    }
}

// Add operations WITHOUT touching shape classes
class AreaCalculator implements ShapeVisitor
{
    public function visitCircle(Circle $c): float      { return M_PI * $c->radius ** 2; }
    public function visitRectangle(Rectangle $r): float { return $r->width * $r->height; }
    public function visitTriangle(Triangle $t): float   { return 0.5 * $t->base * $t->height; }
}

class PerimeterCalculator implements ShapeVisitor
{
    public function visitCircle(Circle $c): float      { return 2 * M_PI * $c->radius; }
    public function visitRectangle(Rectangle $r): float { return 2 * ($r->width + $r->height); }
    public function visitTriangle(Triangle $t): float   { return 0; } // simplified
}

class ShapeSerializer implements ShapeVisitor
{
    public function visitCircle(Circle $c): string      { return "Circle(r={$c->radius})"; }
    public function visitRectangle(Rectangle $r): string { return "Rect({$r->width}×{$r->height})"; }
    public function visitTriangle(Triangle $t): string   { return "Triangle(b={$t->base},h={$t->height})"; }
}

$shapes    = [new Circle(5), new Rectangle(4, 6), new Triangle(3, 4)];
$areaCalc  = new AreaCalculator();
$serializer = new ShapeSerializer();

foreach ($shapes as $shape) {
    echo $serializer->accept($shape) . ": area = " .    // double dispatch
         number_format($shape->accept($areaCalc), 2) . "\n";
}

// Adding new OPERATION: new class implementing ShapeVisitor — shapes unchanged
// Adding new SHAPE: new Shape class + update all visitors — but operations unchanged
```

---

### Union Types + Match — Controlled Ad-hoc Polymorphism

```php
<?php

// PHP 8.0+ — union types let one function handle multiple types safely

function formatValue(int|float|string|bool|null $value): string
{
    return match(true) {
        is_null($value)  => 'NULL',
        is_bool($value)  => $value ? 'TRUE' : 'FALSE',
        is_int($value)   => number_format($value),
        is_float($value) => number_format($value, 2),
        is_string($value) => "\"{$value}\"",
    };
}

echo formatValue(42);        // 42
echo formatValue(3.14159);   // 3.14
echo formatValue("hello");   // "hello"
echo formatValue(true);      // TRUE
echo formatValue(null);      // NULL

// PHP 8.1 — Intersection types for stricter polymorphism
interface Countable   { public function count(): int; }
interface Iterable2   { public function toArray(): array; }

function processCollection(Countable&Iterable2 $collection): void
{
    echo "Count: " . $collection->count() . "\n";
    print_r($collection->toArray());
}
```

---

### Enum-Based Polymorphism (PHP 8.1+)

```php
<?php

enum PaymentStatus: string
{
    case Pending   = 'pending';
    case Completed = 'completed';
    case Failed    = 'failed';
    case Refunded  = 'refunded';

    // Polymorphic behaviour on enum values
    public function label(): string
    {
        return match($this) {
            self::Pending   => '⏳ Pending',
            self::Completed => '✅ Completed',
            self::Failed    => '❌ Failed',
            self::Refunded  => '↩️ Refunded',
        };
    }

    public function isTerminal(): bool
    {
        return match($this) {
            self::Completed, self::Failed, self::Refunded => true,
            self::Pending                                  => false,
        };
    }

    public function canTransitionTo(self $next): bool
    {
        return match($this) {
            self::Pending   => in_array($next, [self::Completed, self::Failed], true),
            self::Completed => $next === self::Refunded,
            self::Failed    => false,
            self::Refunded  => false,
        };
    }
}

// Interface on backed enum (PHP 8.1+)
interface HasColor { public function color(): string; }

enum Priority: int implements HasColor
{
    case Low    = 1;
    case Medium = 2;
    case High   = 3;
    case Critical = 4;

    public function color(): string
    {
        return match($this) {
            self::Low      => '#green',
            self::Medium   => '#yellow',
            self::High     => '#orange',
            self::Critical => '#red',
        };
    }
}

$status = PaymentStatus::Pending;
echo $status->label();                                // ⏳ Pending
echo $status->canTransitionTo(PaymentStatus::Completed) ? "yes\n" : "no\n"; // yes
echo $status->canTransitionTo(PaymentStatus::Refunded) ? "yes\n" : "no\n";  // no

echo Priority::Critical->color(); // #red
```

---

## 5. For the Critics 🔥

### Criticism 1 — Polymorphism Hides What Code Actually Does

**The Argument:**

> "When I see `$animal->makeSound()` — I don't know what actually runs. I have to find all implementors. In a large codebase with 20 implementations of Notification, `$this->notifier->send()` could do anything. Polymorphism trades explicitness for flexibility — and debugging suffers."

**Valid tradeoff — not a design flaw.**
Yes — you lose immediate call-site visibility. You gain: new implementations without changing callers, testability via mocking, and separation of concerns. The tooling answer is: IDE "Find Implementations" (PHPStorm does this perfectly), type declarations that narrow the possibilities, and keeping implementation lists small. The tradeoff is real — but "I can't tell what this does by reading one line" is a cost worth paying for decoupled, extensible systems.

---

### Criticism 2 — Method Overloading via `__call` Is an Anti-Pattern

**The Argument:**

> "PHP's `__call` magic method creates invisible APIs. `$query->whereStatus('active')` doesn't exist as a method — it's intercepted by `__call`. No IDE autocomplete, no static analysis, no discoverability. This is dynamic polymorphism at the cost of every developer tool becoming blind."

**Largely correct.**
`__call` for public-facing APIs is a smell. PHPDoc `@method` annotations partially help IDEs, but static analysis tools (PHPStan, Psalm) cannot fully verify them. The right use of `__call`: internal framework mechanics where the dynamism is justified (Eloquent's query builder, Symfony's service decoration). Application-level business code should never use `__call` — prefer explicit methods or a builder with documented magic.

---

### Criticism 3 — Polymorphism Does Not Eliminate Conditionals

**The Argument:**

> "Every article says 'replace if/elseif with polymorphism.' But you still need the conditional SOMEWHERE — in the factory that creates the right class, in the DI container, in the configuration. Polymorphism moves the conditional, it doesn't eliminate it."

**Completely correct — and this is the point.**
Polymorphism moves conditionals from being **spread across every caller** to being **concentrated in one factory/configuration**. A thousand `if ($type === 'stripe')` blocks across the codebase → one factory that maps type to class. That's not eliminating conditionals — it's organising them. The benefit: changing behaviour means updating ONE place, not hunting through a thousand call sites. The criticism is technically correct but the conclusion (therefore polymorphism is bad) is wrong.

---

### Criticism 4 — Runtime Polymorphism Has Performance Cost

**The Argument:**

> "Virtual dispatch (calling a method through an interface or abstract class) is slower than a direct function call. In tight loops — rendering 10,000 shapes, processing 1M records — the overhead of polymorphic dispatch accumulates. PHP's object model is not zero-cost."

**True at extreme scale — irrelevant in most applications.**
PHP's vtable dispatch overhead is measurable in benchmarks — microseconds per call. In web request handling: the database query, network I/O, and serialization dwarf polymorphism overhead by 1000x. In CPU-bound batch processing: yes, profile it, then optimise the actual bottleneck. Don't sacrifice design clarity for a theoretical performance win you haven't measured. Premature optimisation applies here.

---

### Criticism 5 — The Visitor Pattern Is Proof Polymorphism Has Limits

**The Argument:**

> "Visitor pattern — double dispatch, accept/visit methods, updating every visitor when a new shape is added — is the result of a language that doesn't support multi-methods or pattern matching properly. It's an OOP workaround for a missing language feature. Enums with match expressions (PHP 8.1+) are a cleaner answer."

**Correct — and PHP 8.1+ moved in the right direction.**
`enum` with `match` IS cleaner than Visitor for many cases. Visitor exists to solve "add operations without modifying the hierarchy" — but if the hierarchy is sealed (closed, not extended by third parties), match expressions on enum types are more readable. Visitor shines when the hierarchy IS open for extension by external code. For internal hierarchies, Visitor is often over-engineering.

---

## 6. Quick Reference

```
Types of Polymorphism in PHP:
  Subtype     → extends + implements → most common, runtime dispatch
  Interface   → implements + type hint → most flexible
  Ad-hoc      → __call, union types + match → use with caution
  Parametric  → PHPDoc generics + PHPStan → compile-time only

Rules:
  ✅ Type-hint to interfaces, not concrete classes
  ✅ Let each class define its own behaviour — no instanceof checks
  ✅ One call → many behaviours → no if/elseif chains
  ✅ Use Strategy pattern for runtime algorithm swapping
  ✅ Use Visitor when adding operations to sealed hierarchies
  ✅ Enum + match for finite, well-known sets of variants
  ❌ Don't use instanceof to decide behaviour — add a method instead
  ❌ Don't use __call for public business APIs
  ❌ Don't confuse "many implementations" with "messy design"
```

| Pattern         | Uses Polymorphism Via | When to Use                       |
| --------------- | --------------------- | --------------------------------- |
| Strategy        | Interface injection   | Swappable algorithms              |
| Visitor         | Double dispatch       | Add ops to fixed hierarchy        |
| Template Method | Abstract inheritance  | Fixed skeleton, variable steps    |
| Command         | Interface + dispatch  | Encapsulate and queue actions     |
| Observer        | Interface callbacks   | Event-driven notification         |
| Decorator       | Interface wrapping    | Add behaviour without subclassing |

---

_← [Pillar 3 — Inheritance](./03_inheritance.md) · [Pillar 5 — Additional Concepts →](./05_additional.md)_
