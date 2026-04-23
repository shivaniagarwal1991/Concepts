# **new** vs **init** (Python) vs PHP Object Creation

---

## Core Idea

- Python has **two phases**:

  - `__new__` → creates the object
  - `__init__` → initializes the object

- PHP has **one visible phase**:
  - `__construct` → initializes the object
  - Object creation is handled internally by the engine

---

## Python Model

### Flow

Object Creation → `__new__` → `__init__`

### Explanation

- `__new__`

  - Creates and returns a new object
  - Low-level control
  - Uses `cls`

- `__init__`
  - Initializes object data
  - Uses `self`

---

### Example

```python
class User:
    def __new__(cls, name):
        print("Creating object")
        return super().__new__(cls)

    def __init__(self, name):
        print("Initializing object")
        self.name = name

u = User("Rahul")
```

---

## PHP Model

### Flow

new → (engine creates object) → `__construct`

---

### Explanation

- Object creation is handled by the **PHP engine**
- Developers cannot override creation like `__new__`
- Only initialization is customizable via `__construct`

---

### Example

```python
class User {
    public $name;

    public function __construct($name) {
        echo "Initializing object";
        $this->name = $name;
    }
}

$u = new User("Rahul");
```

---

## Key Differences

| Feature         | Python                   | PHP                       |
| --------------- | ------------------------ | ------------------------- |
| Object creation | `__new__` (customizable) | Engine (not customizable) |
| Initialization  | `__init__`               | `__construct`             |
| Control level   | High                     | Limited                   |
| Flexibility     | Advanced                 | Simpler                   |
| Complexity      | Higher                   | Lower                     |

---

## Does PHP have something like `__new__`?

Direct equivalent: ❌ No

But similar behavior can be achieved using design patterns:

---

### Factory Pattern

```php
class User {
    private function __construct() {}

    public static function create($name) {
        $obj = new self();
        $obj->name = $name;
        return $obj;
    }
}

 Controls object creation indirectly
```

---

### Singleton Pattern

```php
class DB {
    private static $instance = null;

    private function __construct() {}

    public static function getInstance() {
        if (self::$instance === null) {
            self::$instance = new self();
        }
        return self::$instance;
    }
}

 Ensures only one instance exists
```

---

### Clone Control

```php
private function __clone() {}

 Prevents object duplication
```

---

## Design Philosophy Difference

### Python

- Developer has control over object lifecycle
- Flexible and powerful
- Supports advanced patterns

---

### PHP

- Engine manages object creation
- Simpler and easier to use
- Focused on practical web development

---

## Final Understanding

- Python separates **creation and initialization**
- PHP combines creation (engine) + initialization (`__construct`)
- PHP does not expose low-level creation control like `__new__`

---

## One-line Summary

**Python gives full control over object creation (`__new__`) and initialization (`__init__`), while PHP only allows initialization via `__construct` and handles creation internally.**

---

# **new** vs **init** in Python (Deep Dive)

---

## Big Picture

- `__new__` → **creates** the object (allocation phase)
- `__init__` → **initializes** the object (setup phase)

Flow:

Object Creation → `__new__` → `__init__`

---

## Execution Flow

When you write:

User("Rahul")

Internally:

1. `__new__(cls, ...)` is called → returns a new object
2. `__init__(self, ...)` is called → initializes that object

---

## **new** (Object Creation)

### What is it?

- A **special method** that actually **creates and returns a new instance**
- Runs **before `__init__`**

---

### Syntax

```python
class User:
    def __new__(cls, *args, **kwargs):
        print("Creating instance")
        return super().__new__(cls)
```

---

### Example

```python
class User:
    def __new__(cls, name):
        print("In __new__")
        return super().__new__(cls)

    def __init__(self, name):
        print("In __init__")
        self.name = name

u = User("Rahul")
```

---

### Output

In **new**  
In **init**

---

### Key Points

- First method called
- Must return an object
- Uses `cls` (class reference)
- Responsible for memory allocation

---

## **init** (Object Initialization)

### What is it?

- Initializes the object after it is created

---

### Syntax

```python
class User:
    def __init__(self, name):
        self.name = name
```

---

### Example

```python
class User:
    def __init__(self, name):
        self.name = name

u = User("Rahul")
```

---

### Key Points

- Called after `__new__`
- Does NOT create object
- Uses `self` (instance reference)
- Used for setting attributes

---

## **new** vs **init**

| Feature        | **new**              | **init**                |
| -------------- | -------------------- | ----------------------- |
| Purpose        | Create object        | Initialize object       |
| Runs when      | Before object exists | After object is created |
| First argument | `cls`                | `self`                  |
| Return value   | Must return instance | Returns None            |
| Responsibility | Memory allocation    | Set attributes / logic  |

---

## Important Behavior

### If **new** does NOT return an instance

```python
class A:
    def __new__(cls):
        return None

a = A()
```

`__init__` will NOT be called

---

## Real Use Cases of **new**

---

### 1. Singleton Pattern

```python
class DB:
    _instance = None

    def __new__(cls):
        if cls._instance is None:
            cls._instance = super().__new__(cls)
        return cls._instance
```

---

### 2. Immutable Objects (like int, str)

Python uses `__new__` internally for immutable types

---

### 3. Controlling object creation

- Restrict number of instances
- Return existing objects

---

## Common Mistakes

### Using **new** unnecessarily

Most of the time you only need `__init__`

---

### Forgetting to return object in **new**

Causes runtime issues

---

## When to Use What?

| Use Case                  | Method   |
| ------------------------- | -------- |
| Normal object setup       | **init** |
| Custom object creation    | **new**  |
| Singleton / caching       | **new**  |
| Immutable object handling | **new**  |

---

## Mental Model

```text
__new__  → "Make the object"
__init__ → "Prepare the object"
```

---

## After Update

```text
User.count += 1

User.count = 1

u1.count = 1
u2.count = 1
```

Shared across all instances

---

## Access Behavior (Very Important)

### Case 1: Access via Object

```text
print(u1.count) # 1
```

Python checks in this order:

1. Object
2. Then class

---

### Case 2: Override Mistake

```text
u1.count = 5

Now:

u1.count = 5 (instance variable ❗)
u2.count = 1 (class variable)

```

⚠️ This creates a new instance variable, not modify static

---

## Correct Way to Modify Static

```text
User.count += 1
```

Always use the class name

---
