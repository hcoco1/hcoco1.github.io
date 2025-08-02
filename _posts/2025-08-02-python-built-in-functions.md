---
title: Python Built-in Functions
author: hcoco1
date: 2025-07-31 16:10:00 +0800
categories: [Programming, Python]
tags: [python, built-in-function]
render_with_liquid: false
pin: false
description: In Python, built-in functions are functions that are always available for use without needing to import any libraries or modules
---


#### 1. `print()`

```python
print("Hello, world!")  # Outputs: Hello, world!
```

#### 2. `len()`

```python
fruits = ["apple", "banana", "cherry"]
print(len(fruits))  # Outputs: 3
```

#### 3. `type()`

```python
age = 25
print(type(age))  # Outputs: <class 'int'>
```

#### 4. `int()`, `float()`, `str()`

```python
x = "5"
print(int(x))    # Converts to integer: 5
print(float(x))  # Converts to float: 5.0
print(str(10))   # Converts to string: '10'
```

#### 5. `input()`

```python
# name = input("Enter your name: ")
# print("Hello, " + name)
# (You have to run this in a terminal or script to see the interaction)
```

#### 6. `range()`

```python
for i in range(3):
    print(i)  # Outputs: 0 1 2
```

#### 7. `sum()`

```python
numbers = [1, 2, 3, 4]
print(sum(numbers))  # Outputs: 10
```

#### 8. `max()` / `min()`

```python
values = [4, 8, 1, 9]
print(max(values))  # Outputs: 9
print(min(values))  # Outputs: 1
```

#### 9. `abs()`

```python
n = -7
print(abs(n))  # Outputs: 7
```

#### 10. `round()`

```python
pi = 3.14159
print(round(pi, 2))  # Outputs: 3.14
```

#### 11. `sorted()`

```python
nums = [5, 2, 9, 1]
print(sorted(nums))  # Outputs: [1, 2, 5, 9]
```

------


---