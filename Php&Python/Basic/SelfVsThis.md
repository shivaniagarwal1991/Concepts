# 1. Basic Idea

In both languages, the concept is the same:

> **A way for an object to refer to itself**

However, the implementation is different.

---

# Python: `self`

---

## What is it?

`self` = reference to the current object (instance)

---

### Example

```python
class User:
    def __init__(self, name):
        self.name = name

    def show(self):
        print(self.name)


u1 = User("Rahul")

Internally

User.__init__(u1, "Rahul")
```

u1 becomes self

### Use Cases

- Access instance variables
- Use object data inside methods
- Handle multiple objects

---

## Important

### self is mandatory

    Without self:

```python
def show():
    print(name) ❌
```

    Pros (Python)
        - Explicit → clear code
        - No ambiguity
        - Better readability
     Cons (Python)
        - Slightly verbose (extra typing)
        - Can be confusing for beginners

---

## PHP: $this and self

     PHP has two concepts:

- $this → object (instance)
- self → class (static context)

### $this (instance reference)

#### Example

```php
class User {
public $name;

    function setName($name) {
        $this->name = $name;
    }

}
```

- Use Cases
  - Access instance variables
  - Work with object methods

### self (static reference)

#### Example

```php
class User {
public static $count = 0;

    public static function getCount() {
        return self::$count;
    }

}
```

- Use Cases
  - Static variables
  - Static methods
  - Class-level data

---

## Python vs PHP Comparison

| Feature            | Python             | PHP          |
| ------------------ | ------------------ | ------------ |
| Instance reference | `self`             | `$this`      |
| Static reference   | `cls` / class name | `self`       |
| Mandatory          | Yes                | `$this` auto |
| Style              | Explicit           | Implicit     |

---

## Are self and $this reserved keywords?

### Python: self is not a reserved keyword

```python
class User:
    def __init__(abc, name):
        abc.name = name
```

This is valid. But convention is to always use self

### PHP

     $this is a special variable (acts like reserved)

```php
class User {
    function test() {
        echo $this; // current object
    }
}

```

- Cannot rename $this
- Automatically available

---

## `self` vs `$this` (Keyword / Variable Behavior)

| Feature | Python                     | PHP              |
| ------- | -------------------------- | ---------------- |
| `self`  | Not a keyword (convention) | —                |
| `$this` | —                          | Special/reserved |

---
