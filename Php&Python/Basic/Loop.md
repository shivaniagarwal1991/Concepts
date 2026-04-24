# PHP vs Python — All Loops Explained with Examples

> A comprehensive guide comparing loops in PHP and Python with pros, cons, best/worst use cases, and simple examples.

---

## Table of Contents

1. [for Loop](#1-for-loop)
2. [while Loop](#2-while-loop)
3. [do...while Loop](#3-dowhile-loop)
4. [foreach Loop](#4-foreach-loop)
5. [Nested Loops](#5-nested-loops)
6. [break and continue](#6-break-and-continue)
7. [Python-Only Special Features](#7-python-only-special-features)
8. [Final Summary Table](#8-final-summary-table)

---

## 1. `for` Loop

### Concept

Use `for` loop when you **know in advance how many times** the loop should run.

---

### PHP Example

```php
<?php
// Print numbers 1 to 5
for ($i = 1; $i <= 5; $i++) {
    echo "Number: $i \n";
}
?>
```

**Output:**

```
Number: 1
Number: 2
Number: 3
Number: 4
Number: 5
```

---

### Python Example

```python
# Print numbers 1 to 5
for i in range(1, 6):
    print(f"Number: {i}")
```

**Output:**

```
Number: 1
Number: 2
Number: 3
Number: 4
Number: 5
```

---

### Difference

| Point     | PHP                       | Python             |
| --------- | ------------------------- | ------------------ |
| Start     | `$i = 1` written manually | `range(1, 6)` used |
| Increment | `$i++` must be written    | Automatic          |
| Syntax    | Verbose (3 parts)         | Clean and short    |

---

### Pros & Cons

|            | PHP                                                               | Python                                    |
| ---------- | ----------------------------------------------------------------- | ----------------------------------------- |
| ✅ **Pro** | Full C-style control (init, condition, increment) — very flexible | Clean syntax, works directly on iterables |
| ❌ **Con** | Verbose, must manage everything manually                          | Hard to control index without `range()`   |

---

### ✅ Best Use Case — Multiplication Table

```php
// PHP
for ($i = 1; $i <= 10; $i++) {
    echo "2 x $i = " . (2 * $i) . "\n";
}
```

```python
# Python
for i in range(1, 11):
    print(f"2 x {i} = {2 * i}")
```

---

### ❌ Worst Use Case — Iterating Arrays (use foreach instead)

```php
// PHP - Avoid this, foreach is better here
$names = ["Ali", "Sara", "John"];
for ($i = 0; $i < count($names); $i++) {
    echo $names[$i];
}
```

```python
# Python - Avoid this, enumerate() is better here
names = ["Ali", "Sara", "John"]
for i in range(len(names)):
    print(names[i])
```

---

## 2. `while` Loop

### Concept

Use `while` loop when you **do not know** how many times the loop should run — only the condition is known.

---

### PHP Example

```php
<?php
// Keep spending until money runs out
$money = 100;
while ($money > 0) {
    echo "Money left: $money\n";
    $money -= 30;
}
echo "Money finished!";
?>
```

**Output:**

```
Money left: 100
Money left: 70
Money left: 40
Money left: 10
Money finished!
```

---

### Python Example

```python
# Keep spending until money runs out
money = 100
while money > 0:
    print(f"Money left: {money}")
    money -= 30
print("Money finished!")
```

---

### Difference

| Point    | PHP                             | Python               |
| -------- | ------------------------------- | -------------------- |
| Syntax   | `while ($condition)`            | `while condition:`   |
| Variable | `$money` — dollar sign required | `money` — simple     |
| Body     | `{ }` curly braces              | Indentation (spaces) |

---

### Pros & Cons

|            | PHP                                         | Python                          |
| ---------- | ------------------------------------------- | ------------------------------- |
| ✅ **Pro** | Flexible condition-based looping            | Simple and readable, same logic |
| ❌ **Con** | Risk of infinite loop if condition is wrong | Must write `i += 1` (no `i++`)  |

---

### ✅ Best Use Case — User Input Validation

```php
// PHP
$password = "";
while ($password !== "1234") {
    $password = readline("Enter password: ");
}
echo "Login successful!";
```

```python
# Python
password = ""
while password != "1234":
    password = input("Enter password: ")
print("Login successful!")
```

---

### ❌ Worst Use Case — Forgetting to update variable (Infinite Loop Risk!)

```php
// PHP - WRONG! forgot $i++ — infinite loop!
$i = 0;
while ($i < 5) {
    echo $i;
    // $i++ missing 😱
}
```

```python
# Python - WRONG! forgot i += 1 — infinite loop!
i = 0
while i < 5:
    print(i)
    # i += 1 missing 😱
```

---

## 3. `do...while` Loop

### Concept

The loop body **runs at least once**, and then the condition is checked.

---

### PHP Example

```php
<?php
// Ask for PIN — show prompt at least once
$pin = 0;
do {
    echo "Enter PIN: ";
    $pin = 1234; // simulating user input
    echo "You entered: $pin\n";
} while ($pin !== 1234);
echo "PIN correct!";
?>
```

**Output:**

```
Enter PIN:
You entered: 1234
PIN correct!
```

---

### Python Example (Workaround — no native do...while)

```python
# Python does NOT have do...while!
# We use while True + break instead
pin = 0
while True:
    print("Enter PIN: ")
    pin = 1234  # simulating user input
    print(f"You entered: {pin}")
    if pin == 1234:   # condition checked at the END
        break
print("PIN correct!")
```

---

### ⚠️ Key Difference

```
PHP    →  do { ... } while (condition);   ✅ Native support
Python →  while True + break              ⚠️ Workaround only
```

---

### Pros & Cons

|            | PHP                                         | Python                                      |
| ---------- | ------------------------------------------- | ------------------------------------------- |
| ✅ **Pro** | Loop body runs **at least once** guaranteed | `while True` + `break` achieves same result |
| ❌ **Con** | Rarely used, can be confusing               | No native support — workaround looks ugly   |

---

### ✅ Best Use Case — Menu System

```php
// PHP
do {
    echo "1. Start Game\n";
    echo "2. Settings\n";
    echo "3. Exit\n";
    $choice = 1; // user input simulation
} while ($choice != 3);
```

---

### ❌ Worst Use Case — When body should sometimes be skipped

```php
// PHP - WRONG use of do...while
$number = -1;
do {
    echo $number; // prints -1 even though condition is false!
} while ($number > 0);
// Simple while was better here!
```

---

## 4. `foreach` Loop

### Concept

Used to **iterate over every element** in an array or list.

---

### PHP Example — Simple Array

```php
<?php
$fruits = ["Apple", "Mango", "Banana"];

foreach ($fruits as $fruit) {
    echo "Fruit: $fruit\n";
}
?>
```

**Output:**

```
Fruit: Apple
Fruit: Mango
Fruit: Banana
```

---

### PHP Example — Key => Value (Associative Array)

```php
<?php
$student = [
    "name" => "Ali",
    "age"  => 20,
    "city" => "Delhi"
];

foreach ($student as $key => $value) {
    echo "$key: $value\n";
}
?>
```

**Output:**

```
name: Ali
age: 20
city: Delhi
```

---

### Python Example — Simple List

```python
fruits = ["Apple", "Mango", "Banana"]

for fruit in fruits:
    print(f"Fruit: {fruit}")
```

---

### Python Example — Dictionary (Key-Value)

```python
student = {
    "name": "Ali",
    "age":  20,
    "city": "Delhi"
}

for key, value in student.items():
    print(f"{key}: {value}")
```

---

### Python Example — With Index using `enumerate()`

```python
fruits = ["Apple", "Mango", "Banana"]

for index, fruit in enumerate(fruits):
    print(f"{index}: {fruit}")
```

**Output:**

```
0: Apple
1: Mango
2: Banana
```

---

### Difference

| Point     | PHP                        | Python                     |
| --------- | -------------------------- | -------------------------- |
| Keyword   | Separate `foreach` keyword | Same `for` keyword used    |
| Key-Value | `as $key => $value`        | `.items()` method used     |
| Index     | `$key` comes automatically | `enumerate()` must be used |

---

### Pros & Cons

|            | PHP                                          | Python                                                |
| ---------- | -------------------------------------------- | ----------------------------------------------------- |
| ✅ **Pro** | Built-in support for both arrays and objects | Python's `for` acts as `foreach` — clean and powerful |
| ❌ **Con** | Extra keyword (`foreach`) to remember        | Must remember `enumerate()` for index                 |

---

### ✅ Best Use Case — Shopping Cart Total

```php
// PHP
$cart = ["Shirt" => 500, "Pant" => 800, "Shoes" => 1200];
$total = 0;
foreach ($cart as $item => $price) {
    echo "$item: ₹$price\n";
    $total += $price;
}
echo "Total: ₹$total";
```

```python
# Python
cart = {"Shirt": 500, "Pant": 800, "Shoes": 1200}
total = 0
for item, price in cart.items():
    print(f"{item}: ₹{price}")
    total += price
print(f"Total: ₹{total}")
```

---

### ❌ Worst Use Case — When index-based manipulation is needed

```php
// PHP - foreach is bad when you need to modify by index
$nums = [1, 2, 3, 4, 5];
foreach ($nums as $num) {
    $num = $num * 2; // does NOT modify original array!
}
// Use for loop with index instead
```

---

## 5. Nested Loops

### Concept

A loop **inside another loop** — used for 2D data, patterns, grids.

---

### PHP Example — Multiplication Table

```php
<?php
for ($i = 1; $i <= 3; $i++) {
    for ($j = 1; $j <= 3; $j++) {
        echo "$i x $j = " . ($i * $j) . "\t";
    }
    echo "\n";
}
?>
```

**Output:**

```
1x1=1   1x2=2   1x3=3
2x1=2   2x2=4   2x3=6
3x1=3   3x2=6   3x3=9
```

---

### Python Example — Multiplication Table

```python
for i in range(1, 4):
    for j in range(1, 4):
        print(f"{i}x{j}={i*j}", end="\t")
    print()
```

---

### ✅ Best Use Case — Star Pattern

```php
// PHP
for ($i = 1; $i <= 5; $i++) {
    for ($j = 1; $j <= $i; $j++) {
        echo "* ";
    }
    echo "\n";
}
```

```python
# Python
for i in range(1, 6):
    for j in range(1, i + 1):
        print("*", end=" ")
    print()
```

**Output:**

```
*
* *
* * *
* * * *
* * * * *
```

---

### Pros & Cons

|            | PHP                                         | Python                                      |
| ---------- | ------------------------------------------- | ------------------------------------------- |
| ✅ **Pro** | Curly braces make scope visually clear      | Indentation makes structure visually clear  |
| ❌ **Con** | Deep nesting — closing braces get confusing | Deep nesting — indentation becomes too wide |

---

### ❌ Worst Use Case — 3+ levels deep nesting

```python
# Avoid this — break into functions instead
for i in range(5):
    for j in range(5):
        for k in range(5):   # 3 levels — hard to read!
            print(i, j, k)
```

---

## 6. `break` and `continue`

### `break` — Stop the Loop Immediately

```php
// PHP - Find first even number
for ($i = 1; $i <= 10; $i++) {
    if ($i % 2 == 0) {
        echo "First even number: $i";
        break; // found it, stop
    }
}
```

```python
# Python
for i in range(1, 11):
    if i % 2 == 0:
        print(f"First even number: {i}")
        break  # found it, stop
```

**Output:** `First even number: 2`

---

### `continue` — Skip Current Iteration Only

```php
// PHP - Print only odd numbers
for ($i = 1; $i <= 6; $i++) {
    if ($i % 2 == 0) continue; // skip even
    echo "$i\n";
}
```

```python
# Python
for i in range(1, 7):
    if i % 2 == 0:
        continue  # skip even
    print(i)
```

**Output:**

```
1
3
5
```

> ✅ Both PHP and Python use `break` and `continue` in almost the same way!

---

## 7. Python-Only Special Features

### `for...else` — Does NOT exist in PHP!

```python
# Check if "Mango" is in the list
fruits = ["Apple", "Banana", "Cherry"]

for fruit in fruits:
    if fruit == "Mango":
        print("Mango found!")
        break
else:
    # This runs only when loop completes WITHOUT break
    print("Mango not found!")
```

**Output:** `Mango not found!`

> The `else` block runs only if the loop was **never broken** with `break`. This is a unique Python feature — PHP has no equivalent.

---

### List Comprehension — Loop in One Line!

```python
# Traditional loop
squares = []
for i in range(1, 6):
    squares.append(i ** 2)
print(squares)  # [1, 4, 9, 16, 25]

# List Comprehension — same result, one line! ✨
squares = [i ** 2 for i in range(1, 6)]
print(squares)  # [1, 4, 9, 16, 25]
```

---

### List Comprehension with Condition

```python
# Get only even numbers from a list
numbers = [1, 2, 3, 4, 5, 6, 7, 8]
evens = [n for n in numbers if n % 2 == 0]
print(evens)  # [2, 4, 6, 8]
```

> PHP has no direct equivalent of list comprehension. You always need a full loop and `array_push()` or `[]` append.

---

## 8. Final Summary Table

| Loop               | PHP Syntax              | Python Syntax           | When to Use                  |
| ------------------ | ----------------------- | ----------------------- | ---------------------------- |
| `for`              | `for($i=0; $i<5; $i++)` | `for i in range(5):`    | Fixed count iterations       |
| `while`            | `while($x > 0)`         | `while x > 0:`          | Unknown number of iterations |
| `do...while`       | `do { } while()`        | `while True: ... break` | Body must run at least once  |
| `foreach`          | `foreach($arr as $v)`   | `for v in list:`        | Iterating over collections   |
| `for...else`       | ❌ Not available        | `for ... else:`         | Search loops                 |
| List Comprehension | ❌ Not available        | `[x for x in list]`     | Building lists concisely     |

---

## Key Takeaways

- **PHP** is more traditional (C-style) — `for`, `foreach`, `do-while` are all separate keywords
- **Python** is more elegant — a single `for` loop handles what PHP needs both `for` and `foreach` for
- Python's **`for...else`** and **list comprehension** are unique features not found in PHP
- **`while`**, **`break`**, and **`continue`** work almost identically in both languages
- Always avoid deeply nested loops (3+ levels) — break them into separate functions for readability

---
