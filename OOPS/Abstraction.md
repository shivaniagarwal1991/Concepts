# Pillar 2: Abstraction

### Hiding Complexity · Exposing Only What Matters · Interfaces & Abstract Classes

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

> **Abstraction** = Show only **what** something does, hide **how** it does it.

Two mechanisms in PHP:

- **Abstract Classes** — partial blueprint: some done, some must-be-done
- **Interfaces** — pure contract: defines WHAT, never HOW

---

## 2. For a 5-Year-Old 🧸

### The TV Remote Story 📺

When you press the **Volume Up** button on the TV remote — the volume goes up.

You do NOT need to know:

- How the infrared signal is generated
- How the TV's amplifier circuit adjusts the speaker voltage
- How the sound chip decodes the signal
- How the speaker driver translates electrical signals to sound waves

You just know: **press button → volume increases.**

That is **abstraction** — the remote shows you the **WHAT** (volume up button) and hides the **HOW** (all the electronics).

```
TV Remote (Interface)
  ┌──────────────────┐
  │  🔊 VolumeUp()   │  ← you see this
  │  🔉 VolumeDown() │  ← you see this
  │  📺 ChannelUp()  │  ← you see this
  │  ⏻  PowerOff()  │  ← you see this
  └──────────────────┘
         ↑
  What's hidden:  infrared circuits, firmware, amplifier logic, speaker drivers
```

Different TVs (Samsung, LG, Sony) all respond to the same remote — same buttons, completely different insides.

That's the power: **one interface, many different implementations.**

---

## 3. For a Mid-Level Developer 💻

### WITHOUT Abstraction — The Problem ❌

```php
<?php

// No abstraction — every caller knows HOW payment works internally
class OrderService
{
    public function checkout(array $cart, string $paymentType): void
    {
        $total = array_sum(array_column($cart, 'price'));

        // All payment logic directly HERE — no abstraction
        if ($paymentType === 'stripe') {
            // Stripe-specific code — 50 lines of Stripe SDK
            $stripe  = new \Stripe\StripeClient('sk_live_xxx');
            $intent  = $stripe->paymentIntents->create([
                'amount'   => $total * 100,
                'currency' => 'eur',
            ]);
            echo "Stripe charged: €$total\n";

        } elseif ($paymentType === 'paypal') {
            // PayPal-specific code — 50 lines of PayPal SDK
            $paypal = new \PayPal\Api\Payment();
            // ... paypal specific setup ...
            echo "PayPal charged: €$total\n";

        } elseif ($paymentType === 'sepa') {
            // SEPA-specific code — 50 lines of SEPA logic
            echo "SEPA charged: €$total\n";
        }

        // Problems:
        // 1. Adding new payment = modify this class (violates Open/Closed)
        // 2. Testing without real Stripe/PayPal is impossible
        // 3. OrderService knows TOO MUCH — it shouldn't know Stripe SDK at all
        // 4. The if/elseif grows forever
    }
}
```

---

### WITH Abstraction — Interface Approach ✅

```php
<?php

// Step 1: Define the WHAT — not the HOW
interface PaymentGateway
{
    public function charge(float $amount, string $currency): PaymentResult;
    public function refund(string $transactionId, float $amount): bool;
    public function getTransactionId(): string;
}

// Step 2: Value object for result
final class PaymentResult
{
    public function __construct(
        public readonly bool   $success,
        public readonly string $transactionId,
        public readonly string $errorMessage = ''
    ) {}
}

// Step 3: Concrete implementations — each hides ITS OWN HOW
class StripeGateway implements PaymentGateway
{
    public function __construct(private string $apiKey) {}

    public function charge(float $amount, string $currency): PaymentResult
    {
        // All Stripe complexity hidden here — caller never sees it
        try {
            // $stripe = new \Stripe\StripeClient($this->apiKey);
            // $intent = $stripe->paymentIntents->create([...]);
            $txId = 'stripe_' . uniqid();
            return new PaymentResult(true, $txId);
        } catch (\Exception $e) {
            return new PaymentResult(false, '', $e->getMessage());
        }
    }

    public function refund(string $transactionId, float $amount): bool
    {
        // Stripe refund logic hidden here
        return true;
    }

    public function getTransactionId(): string { return 'stripe_' . uniqid(); }
}

class PayPalGateway implements PaymentGateway
{
    public function charge(float $amount, string $currency): PaymentResult
    {
        // All PayPal complexity hidden here
        return new PaymentResult(true, 'paypal_' . uniqid());
    }

    public function refund(string $transactionId, float $amount): bool { return true; }
    public function getTransactionId(): string { return 'paypal_' . uniqid(); }
}

class SepaGateway implements PaymentGateway
{
    public function charge(float $amount, string $currency): PaymentResult
    {
        return new PaymentResult(true, 'sepa_' . uniqid());
    }

    public function refund(string $transactionId, float $amount): bool { return true; }
    public function getTransactionId(): string { return 'sepa_' . uniqid(); }
}

// Step 4: OrderService works with the ABSTRACTION — not the implementation
class OrderService
{
    // Injected — doesn't know OR care which gateway it is
    public function __construct(private PaymentGateway $payment) {}

    public function checkout(array $cart): void
    {
        $total  = array_sum(array_column($cart, 'price'));
        $result = $this->payment->charge($total, 'EUR');

        if (!$result->success) {
            throw new \RuntimeException("Payment failed: {$result->errorMessage}");
        }

        echo "Order complete. Transaction: {$result->transactionId}\n";
    }
}

// Compose with any payment — OrderService NEVER changes
$stripeOrder = new OrderService(new StripeGateway('sk_live_xxx'));
$paypalOrder = new OrderService(new PayPalGateway());

$cart = [['name' => 'Book', 'price' => 29.99]];
$stripeOrder->checkout($cart);
$paypalOrder->checkout($cart);

// Adding Klarna tomorrow: 1 new class, 0 changes to OrderService
class KlarnaGateway implements PaymentGateway { /* ... */ }
$klarnaOrder = new OrderService(new KlarnaGateway());
```

---

### Abstract Class — Partial Abstraction

Use when some behaviour is SHARED but some must be implemented by children.

```php
<?php

abstract class ReportExporter
{
    // Template method — defines the WHAT (algorithm skeleton)
    // Final: children cannot change the order of steps
    final public function export(array $data, string $outputPath): void
    {
        $validated = $this->validate($data);       // shared — implemented here
        $formatted = $this->format($validated);    // abstract — each child defines HOW
        $this->writeToFile($formatted, $outputPath); // shared
        $this->onExportComplete($outputPath);       // hook — optional override
    }

    // SHARED — implemented once, used by all
    protected function validate(array $data): array
    {
        return array_filter($data, fn($row) => !empty($row));
    }

    private function writeToFile(string $content, string $path): void
    {
        file_put_contents($path, $content);
        echo "Written to: $path\n";
    }

    // ABSTRACT — each child must define HOW to format
    abstract protected function format(array $data): string;

    // HOOK — optional override (has a default — do nothing)
    protected function onExportComplete(string $path): void {}
}

class CsvExporter extends ReportExporter
{
    protected function format(array $data): string
    {
        $header = implode(',', array_keys($data[0] ?? []));
        $rows   = array_map(fn($row) => implode(',', $row), $data);
        return $header . "\n" . implode("\n", $rows);
    }
}

class JsonExporter extends ReportExporter
{
    protected function format(array $data): string
    {
        return json_encode($data, JSON_PRETTY_PRINT);
    }

    // Override the hook — send Slack notification after export
    protected function onExportComplete(string $path): void
    {
        echo "Slack: Report exported to $path\n";
    }
}

class HtmlExporter extends ReportExporter
{
    protected function format(array $data): string
    {
        $rows = array_map(function($row) {
            $cells = implode('', array_map(fn($v) => "<td>$v</td>", $row));
            return "<tr>$cells</tr>";
        }, $data);
        return "<table>" . implode('', $rows) . "</table>";
    }
}

$data = [
    ['name' => 'Shivani', 'role' => 'Lead Dev', 'salary' => '95000'],
    ['name' => 'Alex',    'role' => 'Dev',       'salary' => '75000'],
];

(new CsvExporter())->export($data, '/tmp/report.csv');
(new JsonExporter())->export($data, '/tmp/report.json');
(new HtmlExporter())->export($data, '/tmp/report.html');
```

---

### Interface vs Abstract Class — When to Use Which

```php
<?php

// USE INTERFACE when:
// → Multiple unrelated classes need to share a contract
// → No shared implementation exists
// → A class must implement multiple contracts

interface Loggable   { public function log(string $msg): void; }
interface Cacheable  { public function getCacheKey(): string; }
interface Searchable { public function toSearchDocument(): array; }

class Product implements Loggable, Cacheable, Searchable
{
    public function log(string $msg): void          { echo "[Product] $msg\n"; }
    public function getCacheKey(): string           { return "product_{$this->id}"; }
    public function toSearchDocument(): array       { return ['id' => $this->id]; }

    public function __construct(private int $id) {}
}

// USE ABSTRACT CLASS when:
// → Shared implementation exists (DRY)
// → Template Method pattern is needed
// → Children need access to protected shared methods
// → You want to force a specific algorithm skeleton

abstract class BaseRepository
{
    protected \PDO $pdo;

    public function __construct(\PDO $pdo)
    {
        $this->pdo = $pdo;
    }

    // Shared — all repositories can use this
    protected function query(string $sql, array $params = []): array
    {
        $stmt = $this->pdo->prepare($sql);
        $stmt->execute($params);
        return $stmt->fetchAll(\PDO::FETCH_ASSOC);
    }

    // Abstract — each repository defines its own table and mapping
    abstract protected function getTable(): string;
    abstract protected function mapRow(array $row): object;

    public function findAll(): array
    {
        $rows = $this->query("SELECT * FROM {$this->getTable()}");
        return array_map([$this, 'mapRow'], $rows);
    }

    public function findById(int $id): ?object
    {
        $rows = $this->query(
            "SELECT * FROM {$this->getTable()} WHERE id = ?", [$id]
        );
        return $rows ? $this->mapRow($rows[0]) : null;
    }
}

class UserRepository extends BaseRepository
{
    protected function getTable(): string { return 'users'; }

    protected function mapRow(array $row): object
    {
        return new User($row['id'], $row['email'], $row['name']);
    }
}
```

---

## 4. For an Expert Developer 🧠

### Leaky Abstraction — When Abstraction Fails

#### meaning: Leaky abstraction is when an abstraction fails to hide underlying complexity completely, forcing the developer to understand internal details of the system.

| Concept           | Meaning                                                 |
| ----------------- | ------------------------------------------------------- |
| Abstraction       | hide complexity                                         |
| Leaky abstraction | complexity leak(All non-trivial abstractions are leaky) |

- Trivial = बहुत आसान (2+2 = 4)
- Non-trivial = थोड़ा complex / real-world level/ जो simple नहीं है, जिसमें complexity / depth / edge cases हों। (Build a payment system with retries, failures, concurrency)

```php
<?php

// A "leaky" abstraction — the implementation details bleed through the interface

interface Cache
{
    public function get(string $key): mixed;
    public function set(string $key, mixed $value, int $ttl = 3600): void;
    public function delete(string $key): void;
}

class RedisCache implements Cache
{
    public function get(string $key): mixed
    {
        // Redis can only store strings — arrays are serialized
        $value = $this->redis->get($key);
        return $value ? unserialize($value) : null;
    }

    public function set(string $key, mixed $value, int $ttl = 3600): void
    {
        // Redis key length limit: 512MB — but practically ~250 chars
        if (strlen($key) > 250) {
            throw new \InvalidArgumentException("Key too long"); // ← Redis detail leaking!
        }
        $this->redis->setex($key, $ttl, serialize($value));
    }

    public function delete(string $key): void { $this->redis->del($key); }
}

class FileCache implements Cache
{
    public function set(string $key, mixed $value, int $ttl = 3600): void
    {
        // File system key becomes filename — '/' characters break file paths!
        $safeKey = str_replace('/', '_', $key); // ← leaks file system detail
        // ...
    }
}

// Problem: Callers now have to know Redis key length limits
// even though they're programming to the `Cache` interface
// The abstraction has "leaked" its implementation details upward

// Fix: Handle implementation details INSIDE, never expose them
class RedisCache implements Cache
{
    private function normalizeKey(string $key): string
    {
        // Silently handle limits — caller never needs to know
        return strlen($key) > 200 ? md5($key) : $key;
    }

    public function set(string $key, mixed $value, int $ttl = 3600): void
    {
        $this->redis->setex($this->normalizeKey($key), $ttl, serialize($value));
    }
}
```

---

### PSR Interfaces — PHP Community Abstraction Standards

```php
<?php

// PHP-FIG PSR interfaces are the gold standard of PHP abstraction

// PSR-3: Logger Interface
use Psr\Log\LoggerInterface;
use Psr\Log\LogLevel;

class PaymentService
{
    // Programming to PSR-3 interface — not to Monolog, not to any specific logger
    public function __construct(private LoggerInterface $logger) {}

    public function processPayment(float $amount): void
    {
        $this->logger->info('Processing payment', ['amount' => $amount]);
        // Works with Monolog, Symfony Logger, Graylog, CloudWatch — any PSR-3 impl
    }
}

// PSR-11: Container Interface
use Psr\Container\ContainerInterface;

class ServiceFactory
{
    public function __construct(private ContainerInterface $container) {}

    public function getPaymentService(): PaymentService
    {
        return $this->container->get(PaymentService::class);
        // Works with Symfony DI, PHP-DI, Laravel Container — any PSR-11 impl
    }
}

// PSR-7: HTTP Message Interface
use Psr\Http\Message\ServerRequestInterface;
use Psr\Http\Message\ResponseInterface;

// Middleware that works with ANY PSR-7 framework
function authMiddleware(
    ServerRequestInterface $request,
    callable               $next
): ResponseInterface {
    $token = $request->getHeaderLine('Authorization');
    if (empty($token)) {
        // Works with Slim, Laminas, any PSR-7 framework
    }
    return $next($request);
}
```

---

### Multi-Level Abstraction — Layered Architecture

```php
<?php

// Each layer only knows about the layer directly below it
// Each interface is the abstraction boundary between layers

// ── Domain Layer (knows nothing about infrastructure) ──────────────

interface OrderRepositoryInterface    // domain-defined interface
{
    public function save(Order $order): void;
    public function findById(int $id): ?Order;
    public function findByCustomer(int $customerId): array;
}

interface EventPublisherInterface
{
    public function publish(DomainEvent $event): void;
}

class OrderService   // pure domain — no DB, no HTTP, no framework
{
    public function __construct(
        private OrderRepositoryInterface $orders,
        private EventPublisherInterface  $events
    ) {}

    public function placeOrder(array $items, int $customerId): Order
    {
        $order = Order::create($items, $customerId);
        $this->orders->save($order);
        $this->events->publish(new OrderPlaced($order));
        return $order;
    }
}

// ── Infrastructure Layer (implements domain interfaces) ────────────

class DoctrineOrderRepository implements OrderRepositoryInterface
{
    // All Doctrine / SQL complexity hidden here
    public function save(Order $order): void { /* ORM persist */ }
    public function findById(int $id): ?Order { /* ORM find */ return null; }
    public function findByCustomer(int $customerId): array { return []; }
}

class RabbitMQEventPublisher implements EventPublisherInterface
{
    // All RabbitMQ complexity hidden here
    public function publish(DomainEvent $event): void { /* AMQP publish */ }
}

// ── Application Layer (wires it all together) ──────────────────────

$orderService = new OrderService(
    new DoctrineOrderRepository($pdo),
    new RabbitMQEventPublisher($connection)
);

// Domain never touches Doctrine or RabbitMQ — abstraction holds perfectly
```

---

### Type-Safe Abstraction with Generics (PHP 8.x via PHPDoc)

```php
<?php

/**
 * @template T
 */
interface Repository
{
    /**
     * @param int $id
     * @return T|null
     */
    public function findById(int $id): mixed;

    /**
     * @return T[]
     */
    public function findAll(): array;

    /**
     * @param T $entity
     */
    public function save(mixed $entity): void;
}

/**
 * @implements Repository<User>
 */
class UserRepository implements Repository
{
    /** @return User|null */
    public function findById(int $id): ?User { return null; }

    /** @return User[] */
    public function findAll(): array { return []; }

    /** @param User $entity */
    public function save(mixed $entity): void { /* save user */ }
}

// PHPStan and Psalm enforce the generic types at static analysis time
```

---

## 5. For the Critics 🔥

### Criticism 1 — Abstraction Layers Add Indirection Without Value

**The Argument:**

> "Every interface means one more file to open when tracing a bug. `PaymentGateway` → `StripeGateway` → `Stripe\StripeClient` → ... three hops to find the actual logic. Abstraction makes code harder to read, not easier."

**Valid in small codebases. Wrong at scale.**
The indirection IS the value. When StripeGateway changes, nothing outside it changes. When you switch to Adyen — swap one class. The debugging cost (one more hop) is paid once per session. The coupling cost of no abstraction is paid every time you make a change. The tradeoff is: indirection vs coupling. In systems that evolve — indirection wins. In stable systems that never change — flat code wins.

---

### Criticism 2 — Abstract Classes Are Overused as Code-Sharing Mechanisms

**The Argument:**

> "Most abstract classes in PHP codebases are not Template Method patterns — they're just base classes with shared utility methods that nobody wanted to put in a service. Protected helper methods in abstract classes are a code smell: you have tightly coupled inheritance for the sake of sharing three helper functions."

**Correct.**
If an abstract class exists only to share helper methods — extract those to a trait or a standalone service. Abstract classes should express a **stable algorithm with variable steps** (Template Method). If there is no algorithm skeleton — there is no abstract class. The test: could you replace the abstract class with an interface + a trait + constructor injection? If yes — probably should.

---

### Criticism 3 — Abstraction Cannot Handle Impedance Mismatch

**The Argument:**

> "You abstract over payment gateways, but Stripe has features PayPal doesn't (payment intents, SetupIntents for future payments) and vice versa. The abstraction leaks anyway. Your interface is the lowest common denominator — you lose the power of each specific API."

**The strongest criticism against abstraction — and the most honest.**
This is the **impedance mismatch problem**. All gateways can `charge()` — but Stripe's 3DS flow, PayPal's billing agreements, and SEPA's IBAN mandate are gateway-specific. The interface cannot model all of them without either leaking specifics or losing features.

The solution: abstract only what is **genuinely shared**. Add gateway-specific capabilities through extension or separate interfaces. Never force a common denominator that strips the power of the underlying tool.

---

## 6. Quick Reference

```
Interface:
  ✅ Pure contract — WHAT, never HOW
  ✅ Multiple per class
  ✅ No implementation — all abstract
  ✅ Use: unrelated classes sharing a contract
  ❌ No shared code

Abstract Class:
  ✅ Partial blueprint — some done, some must-do
  ✅ One per class (single inheritance)
  ✅ Can have constructor, properties, concrete methods
  ✅ Use: Template Method, shared base logic
  ❌ Cannot be instantiated directly

When to choose:
  Multiple contracts needed           → Interface
  Shared implementation exists        → Abstract Class
  Both needed                         → Interface + Abstract Class
  External standard (PSR, framework)  → Interface (always)
```

| Keyword             | Means                         | Instantiatable |
| ------------------- | ----------------------------- | -------------- |
| `interface`         | Pure contract                 | ❌ Never       |
| `abstract class`    | Partial implementation        | ❌ Never       |
| `abstract function` | Must-implement method         | —              |
| `final`             | Cannot be overridden/extended | —              |

---

_← [Pillar 1 — Encapsulation](./01_encapsulation.md) · [Pillar 3 — Inheritance →](./03_inheritance.md)_
