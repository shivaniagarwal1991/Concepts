# Variables in PHP & Python (Complete Guide)

- PHP & Python are dynamically typed

---

# What is a Variable?

A variable is a **container used to store data**. Variable = data container

### Example:

```python
x = 10 (store a number)
name = "Rahul" (store a string)
```

# What does a Variable do?

A variable:

- Stores data
- Allows reuse of values
- Helps in calculations
- Makes programs dynamic

Variable Types in PHP

### PHP is a dynamically typed language.

    Supported Types:
      1) int
      2) float
      3) string
      4) bool
      5) array
      6) object
      7) null

\*\* PHP uses a flexible array type for both indexed and associative data.

### Python is also dynamically typed.

    Supported Types:
      1) int
      2) float
      3) str
      4) bool
      5) list  (mutable, dynamic data,slow) (indexed array: ex [1,2,3])
      6) tuple (immutable, fixed data,faster) (ex: (1,2,3))
      7) dict (associative array: ex {"name":"test", "age":"2"})
      8) set
      9) object

# Why Python Separates List and Dictionary?

- Python focuses on clarity and readability
- Each structure has a specific purpose

# Pros of Using Variables

- Improves readability
- Enables reusability
- Makes code dynamic
- Easier debugging

# Cons of Using Variables

- Uses memory
- Can introduce bugs if misused
- Scope issues (global vs local)

# What if Variables Did Not Exist?

### Pros (rare cases):

- Slightly simpler for very small programs

### Cons:

- Code becomes hard to read
- No reusability
- Difficult to maintain
- Debugging becomes very hard

# Variable vs Constant & Python

---

## 1. Variable vs Constant

| Feature      | Variable       | Constant          |
| ------------ | -------------- | ----------------- |
| Value Change | Can be changed | Cannot be changed |
| Use          | Dynamic data   | Fixed value       |
| Example      | `x = 10`       | `PI = 3.14`       |

---

## 2. Why Python Avoided Strict Types?

### Python Philosophy

- Simplicity
- Readability
- Developer productivity

---

### Therefore, Python focuses on:

- Less boilerplate code
- Less syntax complexity

# NULL vs None (PHP vs Python)

---

## Comparison

| Language | Null Value | Type     |
| -------- | ---------- | -------- |
| PHP      | `NULL`     | null     |
| Python   | `None`     | NoneType |

---

## Why Python uses `None` instead of `null`?

---

### Design Reasons

- Everything in Python is an object
- `None` is a **singleton object** (only one instance exists)
- Cleaner and more consistent design

---

## Behavior Difference

### PHP (Loose Comparison)

```php
NULL == 0   // true (dangerous)
NULL == ""  // true

```

### Python (Strict Comparison)

None == 0 # False
None == "" # False
