# Constructor in Python & PHP

---

## What is a Constructor?

A constructor is a special method that runs automatically when an object is created

---

### Purpose

- Initialize object data
- Set default values
- Run setup logic

---

# Python Constructor

---

## Syntax

```python
class User:
    def __init__(self, name):
        self.name = name
```

````

---

## Example

```python
class User:
    def __init__(self, name):
        self.name = name

u = User("Rahul")
print(u.name)  # Rahul
```

---

## How it works

User("Rahul")

Internally:

User.**init**(object, "Rahul")

`self` = object

---

## Key Points (Python)

- Method name: `__init__`
- Runs automatically
- Must include `self`
- Can have parameters

---

# PHP Constructor

---

## Syntax

```php
class User {
    public function __construct($name) {
        $this->name = $name;
    }
}
```

---

## Example

```php
class User {
    public $name;

    public function __construct($name) {
        $this->name = $name;
    }
}

$u = new User("Rahul");
echo $u->name; // Rahul
```

---

## How it works

new User("Rahul");

Automatically calls:

\_\_construct("Rahul")

`$this` = object

---

## Key Points (PHP)

- Method name: `__construct`
- Runs automatically
- `$this` is available
- Can take parameters

---

# Python vs PHP Constructor

| Feature             | Python     | PHP           |
| ------------------- | ---------- | ------------- |
| Name                | `__init__` | `__construct` |
| Object reference    | `self`     | `$this`       |
| Mandatory reference | Yes        | Auto          |
| Style               | Explicit   | Implicit      |

---
````
