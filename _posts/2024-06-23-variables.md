---
title: Understanding Variables 
author: hcoco1
date: 2024-02-03 14:10:00 +0800
categories: [Programming, Languages]
tags: [javascript, python]
render_with_liquid: false
---




A variable declaration is a statement in a computer programming language that specifies a variable's name and type, but does not assign it a value. The declaration tells the compiler that the variable exists in the program and where it is located. It also allows the program to reserve memory for storing the variable's values




## What is a Variable?

A variable is a fundamental concept in programming, acting as a storage location in a computer's memory that holds a value. This value can be changed during the execution of a program, making variables essential for dynamic data handling.

### Characteristics of Variables:

1. **Name**: Identifies the variable.
2. **Type**: Determines the kind of data the variable can hold.
3. **Value**: The actual data stored in the variable.
4. **Scope**: The region of the program where the variable is accessible.
5. **Lifetime**: The duration for which the variable exists in memory.

## Variable Declaration

Variable declaration is the process of defining a variable, specifying its name, and often assigning an initial value. Different programming languages have different syntax and rules for declaring variables.

## Variables in JavaScript

### Declaring Variables

In modern JavaScript, variables can be declared using `var`, `let`, and `const`. Each has different characteristics and scope rules.

#### `var`

- **Scope**: Function-scoped.
- **Reassignable**: Yes.
- **Hoisting**: Yes.

```javascript
var x = 10;
console.log(x); // 10
```

#### `let`

- **Scope**: Block-scoped.
- **Reassignable**: Yes.
- **Hoisting**: No.

```javascript
let y = 20;
console.log(y); // 20
```

#### `const`

- **Scope**: Block-scoped.
- **Reassignable**: No.
- **Hoisting**: No.

```javascript
const z = 30;
console.log(z); // 30
```

### Variable Types

JavaScript supports several types of variables:

1. **Number**
    ```javascript
    let age = 25;
    ```
2. **String**
    ```javascript
    let name = "John";
    ```
3. **Boolean**
    ```javascript
    let isActive = true;
    ```
4. **Null**
    ```javascript
    let empty = null;
    ```
5. **Undefined**
    ```javascript
    let notAssigned;
    ```
6. **Object**
    ```javascript
    let person = { firstName: "John", lastName: "Doe" };
    ```
7. **Array**
    ```javascript
    let numbers = [1, 2, 3];
    ```

### Scope and Hoisting

- **Global Scope**: Variables declared outside any function.
    ```javascript
    var globalVar = "I'm global";
    ```

- **Function Scope**: Variables declared inside a function.
    ```javascript
    function myFunc() {
        var funcVar = "I'm function-scoped";
    }
    ```

- **Block Scope**: Variables declared with `let` or `const` inside a block.
    ```javascript
    if (true) {
        let blockVar = "I'm block-scoped";
    }
    ```

**Hoisting**: JavaScript's behavior of moving declarations to the top.
```javascript
console.log(hoistedVar); // undefined
var hoistedVar = "Hoisted!";
```

## Variables in Python

### Declaring Variables

In Python, variables are dynamically typed, meaning you do not need to declare the type explicitly. The type is inferred from the value assigned.

```python
x = 10
print(x)  # 10
```

### Variable Types

Python supports several types of variables:

1. **Integer**
    ```python
    age = 25
    ```
2. **Float**
    ```python
    price = 99.99
    ```
3. **String**
    ```python
    name = "John"
    ```
4. **Boolean**
    ```python
    is_active = True
    ```
5. **None**
    ```python
    empty = None
    ```
6. **List**
    ```python
    numbers = [1, 2, 3]
    ```
7. **Tuple**
    ```python
    coordinates = (10.0, 20.0)
    ```
8. **Set**
    ```python
    unique_numbers = {1, 2, 3}
    ```
9. **Dictionary**
    ```python
    person = {"first_name": "John", "last_name": "Doe"}
    ```

### Scope

- **Global Scope**: Variables declared at the top level.
    ```python
    global_var = "I'm global"
    ```

- **Local Scope**: Variables declared inside a function.
    ```python
    def my_func():
        local_var = "I'm local"
    ```

### Reassigning Variables

Python allows variables to be reassigned to different types.
```python
x = 10
print(x)  # 10
x = "Hello"
print(x)  # Hello
```

### Constants

By convention, constants are declared using uppercase letters.
```python
PI = 3.14159
```

### Type Hints

Python 3.5+ supports type hints for better code clarity.
```python
x: int = 10
y: str = "Hello"
def add(a: int, b: int) -> int:
    return a + b
```

## Summary

Variables are essential for storing and manipulating data in programming. JavaScript offers `var`, `let`, and `const` for variable declaration with different scoping rules, while Python uses dynamic typing and type hints for improved readability. Understanding these concepts is crucial for effective coding in both languages.


## Videos

<iframe width="560" height="315" src="https://www.youtube.com/embed/nbX0MIV7-Ek?si=No3HAUWgQFzGonGy" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

---

<iframe width="560" height="315" src="https://www.youtube.com/embed/pJ_oKVFHMK0?si=za-d4HYp-sR1R7nl" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

---


<iframe width="560" height="315" src="https://www.youtube.com/embed/dzEieWaOJE0?si=61yNrfwG-eaoO5La" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>