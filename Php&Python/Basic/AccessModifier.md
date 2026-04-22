# Access Modifiers: PHP vs Python

---

## Overview

- **PHP → Strict access control**
- **Python → Convention-based access**

---

## What are Access Modifiers?

Access modifiers control **who can access variables and methods** in a class.

---

## Comparison Table

| Feature     | PHP         | Python                  |
| ----------- | ----------- | ----------------------- |
| Public      | Yes         | Default                 |
| Protected   | Strict      | Convention (`_var`)     |
| Private     | Strict      | Name mangling (`__var`) |
| Enforcement | Strong      | Weak                    |
| Philosophy  | Restriction | Trust                   |

---

## Why Python Does NOT Enforce Strict Access?

### Python Philosophy

> “We are all consenting adults here”

---

### Meaning:

- Developers are trusted to follow conventions
- Unnecessary restrictions are avoided

---

## Deep Concept: Name Mangling (Python)

Private variables in Python are internally renamed:

```text
__password → _ClassName__password
```

### Purpose:

- Prevent accidental override
- Improve inheritance safety
