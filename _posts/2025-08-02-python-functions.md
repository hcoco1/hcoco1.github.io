---
title: Python Functions
author: hcoco1
date: 2025-07-31 16:10:00 +0800
categories: [Programming, Python]
tags: [python, function]
render_with_liquid: false
pin: false
description: A function is a reusable block of code that performs a specific task. It helps you organize your code, avoid repetition, and make programs more readable and modular.

---


## ✨ Benefits of Using Functions

- Write code once, reuse it anywhere
- Makes debugging and testing easier
- Improves readability and structure
- Breaks complex problems into smaller steps

------

## 🧱 Structure of a Python Function

```python
def function_name(parameters):
    # code block
    return result  # optional
```

------

## ✅ Example 1: Basic Function (No Parameters)

```python
def greet():
    print("Hello!")
    
greet()  # Call the function
```

📤 Output:

```
Hello!
```

------

## ✅ Example 2: Function With Parameters

```python
def greet(name):
    print("Hello, " + name + "!")

greet("Alice")
greet("Ivan")
```

📤 Output:

```
Hello, Alice!
Hello, Ivan!
```

------

## ✅ Example 3: Function With Return Value

```python
def add(a, b):
    return a + b

result = add(3, 5)
print("The sum is:", result)
```

📤 Output:

```
The sum is: 8
```

------

## 🔁 Example 4: Using a Function in a Loop

```python
def square(x):
    return x * x

for i in range(5):
    print(square(i))
```

📤 Output:

```
0
1
4
9
16
```

------

## 🧠 Notes

- Use `def` to **define** a function.
- Use `return` to **send back** a result.
- Functions can take **0 or more arguments**.
- You can call a function **as many times as needed**.

------

## 🛠️ Optional: Default Parameter Values

```python
def greet(name="friend"):
    print("Hello, " + name)

greet()         # Uses default
greet("David")  # Overrides default
```

📤 Output:

```
Hello, friend
Hello, David
```

------

