# Static vs Instance (Deep Understanding)

---

## Core Concept

- **Instance → object-specific**
- **Static → shared across objects but limited to class**

---

## What does “Shared” mean?

Shared = a single value or memory location used by **all objects**

---

## Example (Python)

### Static (Shared)

```python
class User:
    count = 0   # static (shared)

u1 = User()
u2 = User()

User.count += 1
```

Result:

```python
u1.count = 1
u2.count = 1
```

Both objects see the same value → this is shared

- in Python, class variables act like static variables by default
- in Python, no static keyword. instead: Class variable = static-like behavior

---

### Instance Case

```python
class User:
    def __init__(self):
        self.count = 0

```

Result:

```python
u1.count = 0
u2.count = 0 (separate)
```

Different memory → not shared

---

## What is an Instance?

Instance = an object (actual copy of a class)

---

## Example (Python)

```python
class User:
    def __init__(self, name):
        self.name = name

u1 = User("Rahul")
u2 = User("Amit")
```

Each object has its own data:

```python
u1.name = Rahul
u2.name = Amit
```

## Example (PHP)

```php
class User {
    public $name;
}

$u1 = new User();
$u2 = new User();
```

---

## What is Static?

Static = class-level data (shared across all objects)

---

## Example (Python)

```python
class User:
    count = 0

    def __init__(self):
        User.count += 1
```

## Example (PHP)

```php
class User {
    public static $count = 0;
}
```

---

# Static vs Instance (Deep Difference with Python & PHP)

---

## Deep Difference

| Feature    | Instance     | Static             |
| ---------- | ------------ | ------------------ |
| Belongs to | Object       | Class              |
| Memory     | Separate     | Shared             |
| Access     | self / $this | class name / self  |
| Use        | User data    | Global/shared data |

---

## Memory-Level Understanding

### Instance

```text
Object1 → name = Rahul
Object2 → name = Amit
```

Separate memory

### Static

```text
Class → count = 2
```

    Shared memory

---

## Example (Python)

### Instance

```python
class User:
    def __init__(self, name):
        self.name = name

u1 = User("Rahul")
u2 = User("Amit")

print(u1.name)  # Rahul
print(u2.name)  # Amit
```

### Static

```python
class User:
    count = 0

    def __init__(self):
        User.count += 1

u1 = User()
u2 = User()

print(User.count)  # 2
```

## Example (PHP)

### Instance

```php
class User {
    public $name;
}

$u1 = new User();
$u1->name = "Rahul";

$u2 = new User();
$u2->name = "Amit";

echo $u1->name; // Rahul
echo $u2->name; // Amit
```

### Static

```php
class User {
    public static $count = 0;

    public function __construct() {
        self::$count++;
    }
}

$u1 = new User();
$u2 = new User();

echo User::$count; // 2
```

---

## Are Static Variables Thread-safe?

    Default: Not thread-safe

### Why?

    Multiple threads access the same variable:

```text
Thread1 → count++
Thread2 → count++
```

    This can cause a race condition

- Solutions:
  - Locks / Mutex
  - Atomic operations

---

### When Should You Use Static?

- Use When:
  - Shared data is required
  - Constant values
  - Counters

---

## Global vs Static

## Core Idea

- **Global → accessible everywhere (program-level)(system-wide shared state)**
- **Static → shared within a class (class-level)(class-scoped shared state)**

---

# What is Global?

    -  A **global variable** is defined outside functions/classes
    -  It can be accessed across different parts of the program

---

## Python Global

```python
x = 10  # global variable

def show():
    global x
    x = 20

show()
print(x)  # 20
```

- Important: Use global keyword to modify it inside a function

## PHP Global

```php
$x = 10;

function test() {
   global $x;
   $x = 20;
}

test();
echo $x; // 20
```

---

# What is Static?

    -  A static variable belongs to the class
    -  It is shared across all objects

---

## Python Static

```python
class User:
    count = 0  # static

    def __init__(self):
        User.count += 1

u1 = User()
u2 = User()

print(User.count)  # 2
```

- Important: Use global keyword to modify it inside a function

## PHP Static

```php
class User {
    public static $count = 0;

    public function __construct() {
        self::$count++;
    }
}

$u1 = new User();
$u2 = new User();

echo User::$count; // 2
```

---

## Deep Comparison

| Feature    | Global                    | Static                 |
| ---------- | ------------------------- | ---------------------- |
| Scope      | Entire program            | Class only             |
| Belongs to | Program                   | Class                  |
| Access     | Direct / `global` keyword | Class name / `self`    |
| Lifetime   | Entire program runtime    | Entire program runtime |
| Use case   | Shared config, constants  | Counters, class data   |

---
