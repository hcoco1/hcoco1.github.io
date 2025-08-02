---
title: Study Guide Module 2
author: hcoco1
date: 2025-07-31 17:10:00 +0800
categories: [Courses, Crash Course on Python]
tags: [python, quiz-2]
render_with_liquid: false
pin: false
description: Study Guide Module 2 Graded Quiz
---

## 🧠 Knowledge Checklist

### ✅ Variables & Expressions

- Assign values to variables using `=`
- Use arithmetic operators: `+`, `-`, `*`, `/`, `//`, `%`, `**`
- Predict the output of expressions

### ✅ Functions

- Define a function using `def`
- Pass arguments (parameters) to functions
- Use `return` to send values back
- Call functions inside `print()` or other functions

```python
def add(a, b):
    return a + b

print(add(3, 4))  # Output: 7
```

------

### ✅ Conditionals

- Use `if`, `elif`, and `else` to control program flow
- Each `if`/`elif` **must end with a colon** (`:`)
- Code under each block must be **indented**
- Use comparison operators to test values

### 🔸 Comparison Operators:

| Symbol | Meaning          | Example  | Result |
| ------ | ---------------- | -------- | ------ |
| `==`   | Equal to         | `5 == 5` | `True` |
| `!=`   | Not equal to     | `5 != 3` | `True` |
| `>`    | Greater than     | `6 > 2`  | `True` |
| `<`    | Less than        | `1 < 5`  | `True` |
| `>=`   | Greater or equal | `7 >= 7` | `True` |
| `<=`   | Less or equal    | `3 <= 8` | `True` |

------

### 🔸 Logical Operators:

| Operator | Meaning                    | Example            | Result |
| -------- | -------------------------- | ------------------ | ------ |
| `and`    | True if **both** are true  | `5 > 3 and 6 < 10` | `True` |
| `or`     | True if **either** is true | `5 > 10 or 6 < 10` | `True` |
| `not`    | Inverts a Boolean          | `not 5 > 10`       | `True` |

------

### 🔸 Alphabetizing Strings

- Uses Unicode values for comparison
- `"a" < "b"` → `True`
- `"Z" < "a"` → `True` (because uppercase letters have smaller Unicode)

------

### 🔸 Floor Division `//` and Modulo `%`

| Operator | Meaning                        | Example  | Result |
| -------- | ------------------------------ | -------- | ------ |
| `//`     | Floor division (drops decimal) | `7 // 2` | `3`    |
| `%`      | Modulo (gives the remainder)   | `7 % 2`  | `1`    |

------

### ✅ Complex Conditionals in `if-elif-else`

You can use multiple conditions with logical operators:

```python
if x > 5 and x < 10:
    print("Between 5 and 10")
elif x == 5 or x == 10:
    print("Exactly 5 or 10")
else:
    print("Outside range")
```

------

## 🛠 Coding Skills

### 🧪 Skill Group 1 – Functions + Conditionals

- Use `def` to define a function
- Use comparison operators to check conditions
- Use `return` to send back a result

```python
def task_reminder(time_as_string):
    if time_as_string == "08:00":
        return "Time to eat!"
    elif time_as_string == "12:00":
        return "Time for lunch!"
    else:
        return "Keep working."
```

------

### 🧪 Skill Group 2 – Predict Output

- Understand what functions return
- Understand how nested functions evaluate

```python
def product(a, b):
    return a * b

print(product(product(2,4), product(3,5)))
# 2*4 = 8, 3*5 = 15, then 8*15 = 120
```

------

### 🧪 Skill Group 3 – Complex Conditions

```python
def get_remainder(x, y):
    if x == 0 or y == 0 or x == y:
        remainder = 0
    else:
        remainder = (x % y) / y
    return remainder

print(get_remainder(10, 3))  # 10 % 3 = 1 → 1 / 3 = 0.333...
```

------

## ⚠️ Syntax Reminders

### Common Mistakes to Avoid:

- ❌ Misspelled keywords: (`retrun`, `fucntion`, etc.)
- ❌ Missing or wrong colons: (`if x == 5` ← needs `:`)
- ❌ Wrong indentation
- ❌ Wrong quote types (`‘smart quotes’` instead of `'` or `"`)
- ❌ Case sensitivity (`Print` vs `print`)
- ❌ Using parentheses or brackets incorrectly

------

### ✅ Best Practices:

- Use meaningful variable names
- Add comments for clarity
- Keep code readable and consistent
- Write **self-documenting code**: code that explains itself without extra comments

------

