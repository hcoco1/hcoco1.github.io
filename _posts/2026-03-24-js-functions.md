---
title: Javascript Functions
author: hcoco1
date: 2026-03-24 16:10:00 +0800
categories: [Programming, Javascript]
tags: [functions]
render_with_liquid: false
pin: false
description: A function is a reusable block of code that performs a specific task.

---

## JavaScript Functions

### 1. Function Declaration

The classic way to define a function. It's **hoisted**, meaning you can call it before it's defined.

```javascript
function greet(name) {
  return `Hello, ${name}!`;
}

console.log(greet("Alice")); // "Hello, Alice!"
```

---

### 2. Function Expression

A function assigned to a variable. Not hoisted — must be defined before use.

```javascript
const square = function(n) {
  return n * n;
};

console.log(square(5)); // 25
```

---

### 3. Arrow Functions

A shorter syntax introduced in ES6. Great for concise logic and callbacks.

```javascript
const add = (a, b) => a + b;

console.log(add(3, 4)); // 7
```

If the body has multiple lines, use curly braces and `return`:

```javascript
const multiply = (a, b) => {
  const result = a * b;
  return result;
};
```

---

### 4. Default Parameters

Provide fallback values when arguments aren't passed.

```javascript
function greet(name = "stranger") {
  return `Hi, ${name}!`;
}

console.log(greet());         // "Hi, stranger!"
console.log(greet("Bob"));    // "Hi, Bob!"
```

---

### 5. Rest Parameters

Collect any number of arguments into an array.

```javascript
function sum(...numbers) {
  return numbers.reduce((total, n) => total + n, 0);
}

console.log(sum(1, 2, 3, 4)); // 10
```

---

### 6. Callback Functions

A function passed as an argument to another function.

```javascript
function doMath(a, b, operation) {
  return operation(a, b);
}

const result = doMath(10, 5, (a, b) => a - b);
console.log(result); // 5
```

---

### 7. Higher-Order Functions

Functions that return other functions — a cornerstone of functional programming.

```javascript
function multiplier(factor) {
  return (number) => number * factor;
}

const double = multiplier(2);
const triple = multiplier(3);

console.log(double(6)); // 12
console.log(triple(6)); // 18
```

---

### Key Differences at a Glance

| Feature | Declaration | Expression | Arrow |
|---|---|---|---|
| Hoisted | ✅ Yes | ❌ No | ❌ No |
| Own `this` | ✅ Yes | ✅ Yes | ❌ No |
| Best for | Named utilities | Assigned logic | Short callbacks |

Arrow functions don't have their own `this`, which makes them ideal for use inside methods and callbacks but not as object methods or constructors.


