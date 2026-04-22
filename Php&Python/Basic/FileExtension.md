# PHP & Python: Questions & Answers

This document covers basic concepts around PHP and Python file extensions, execution, and behavior.

---

## Extension = Label on a file

- `.php` → “Run me with PHP”
- `.py` → “Run me with Python”

---

## What is a file extension?

A file extension is the part after the `.` in a filename.

### Examples:

- `index.php`

  - Filename: `index`
  - Extension: `.php`

- `index.py`
  - Filename: `index`
  - Extension: `.py`

---

## What does a file extension do?

It tells the **operating system (OS)** how to open and execute the file.

---

## Why do we need it?

So the system can understand what type of file it is:

- `.php` → Server-side web script
- `.py` → Python script

---

## Execution Routing

- `.php` → PHP interpreter
- `.py` → Python interpreter

---

## What happens without an extension?

The OS does not know:

- What the file is
- How to execute it

---

## ❗ Important

File extensions are not technically required.

You can rename a file like: test.xyz

But in this case, you must manually specify the interpreter.
And still run it as Python like: python test.xyz

---

## Why do programming languages have different extensions?

Different extensions provide:

- Unique identity
- Execution rules

### Examples:

- `.py` → Python interpreter
- `.php` → PHP engine
- `.js` → JavaScript engine
- `.java` → Java compiler

---

## What can these files contain?

### `.py` (Python file)

Contains:

- Python code
- Functions
- Classes
- Scripts

---

### `.php` (PHP file)

Contains:

- PHP code
- HTML + PHP mixed content

---

## How to identify a file?

- By extension (`.py`, `.php`)
- By content (syntax)
- By interpreter used

---

## What does a file need to represent that extension?

### Python (`.py`)

✔ Must contain valid Python syntax

---

### PHP (`.php`)

✔ Must contain PHP tags:

```php
<?php
echo "Hello";
?>

## Basic conditions to run the file

### PHP file
Cannot run directly in a browser as raw code
Requires a server (e.g., Apache, Nginx)

### Python file
Cannot be executed directly by a browser
Requires a Python interpreter
```
