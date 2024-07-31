---
title: JavaScript Control Structures
author: hcoco1
date: 2024-07-05 14:10:00 +0800
categories: [Programming, Languages]
tags: [javascript]
render_with_liquid: false
---

JavaScript, often abbreviated JS, is a programming language that is one of the core technologies of the World Wide Web, alongside HTML and CSS. It lets us add interactivity to pages e.g. you might have seen sliders, alerts, click interactions, popups, etc on different websites — all of that is built using JavaScript...



## Conditional Statements 

## Explanation:
Conditional statements in JavaScript are used to perform different actions based on different conditions:

- **if**: Executes a block of code if a specified condition is true.
- **else**: Executes a block of code if the same condition is false.
- **else if**: Specifies a new condition to test if the first condition is false.
- **switch**: Specifies many alternative blocks of code to be executed.

## Question 7: `if`, `else`, `else if`

Write a function that takes a number as input and logs whether the number is positive, negative, or zero. Additionally, add more complexity by checking if the number is even or odd.

```javascript
function checkNumber(num) {
    if (num > 0) {
        console.log("Positive");
        if (num % 2 === 0) {
            console.log("Even");
        } else {
            console.log("Odd");
        }
    } else if (num < 0) {
        console.log("Negative");
        if (num % 2 === 0) {
            console.log("Even");
        } else {
            console.log("Odd");
        }
    } else {
        console.log("Zero");
    }
}

// Testing the function
checkNumber(5);   // Positive, Odd
checkNumber(-3);  // Negative, Odd
checkNumber(0);   // Zero
checkNumber(4);   // Positive, Even
checkNumber(-8);  // Negative, Even
```

## Question 8: `switch`

Write a function that takes a grade (A, B, C, D, F) as input and logs a corresponding message. 

```javascript
function checkGrade(grade) {
    switch (grade) {
        case 'A':
            console.log("Excellent!");
            break;
        case 'B':
            console.log("Well done!");
            break;
        case 'C':
            console.log("Good");
            break;
        case 'D':
            console.log("Needs Improvement");
            break;
        case 'F':
            console.log("Fail");
            break;
        default:
            console.log("Invalid grade");
    }
}

// Testing the function
checkGrade('A');  // Excellent!
checkGrade('B');  // Well done!
checkGrade('C');  // Good
checkGrade('D');  // Needs Improvement
checkGrade('F');  // Fail
checkGrade('G');  // Invalid grade
```

## Extended Example: Combining Conditional Statements and Loops

Write a function that takes an array of numbers as input and logs whether each number is positive, negative, or zero, and whether it is even or odd. Use both `if`, `else if`, and `switch` statements.

```javascript
function analyzeNumbers(numbers) {
    for (let i = 0; i < numbers.length; i++) {
        let num = numbers[i];
        if (num > 0) {
            console.log(num + " is Positive");
            if (num % 2 === 0) {
                console.log(num + " is Even");
            } else {
                console.log(num + " is Odd");
            }
        } else if (num < 0) {
            console.log(num + " is Negative");
            if (num % 2 === 0) {
                console.log(num + " is Even");
            } else {
                console.log(num + " is Odd");
            }
        } else {
            console.log(num + " is Zero");
        }
        
        switch (true) {
            case (num > 0 && num % 2 === 0):
                console.log(num + " is a Positive Even number");
                break;
            case (num > 0 && num % 2 !== 0):
                console.log(num + " is a Positive Odd number");
                break;
            case (num < 0 && num % 2 === 0):
                console.log(num + " is a Negative Even number");
                break;
            case (num < 0 && num % 2 !== 0):
                console.log(num + " is a Negative Odd number");
                break;
            case (num === 0):
                console.log(num + " is Zero");
                break;
            default:
                console.log("Invalid number");
        }
    }
}

// Testing the function
analyzeNumbers([5, -3, 0, 4, -8]);
/*
Output:
5 is Positive
5 is Odd
5 is a Positive Odd number
-3 is Negative
-3 is Odd
-3 is a Negative Odd number
0 is Zero
0 is Zero
4 is Positive
4 is Even
4 is a Positive Even number
-8 is Negative
-8 is Even
-8 is a Negative Even number
*/
```

These examples cover the basics of conditional statements (`if`, `else`, `else if`, and `switch`) and demonstrate how they can be used in different scenarios. The extended example combines these concepts with loops to analyze an array of numbers, providing a more comprehensive understanding of how to use conditional statements in JavaScript.




## 3. Functions

### 3.1 Function Declaration and Invocation

#### Explanation:
A function is a block of code designed to perform a particular task. It is executed when "called" (invoked).
- **Function Declaration**: Defines a function using the `function` keyword.
- **Function Invocation**: Executes the function by calling it with parentheses.

#### Question 11: Function Declaration and Invocation
Declare a function that takes two numbers as arguments and returns their sum. Invoke the function with two numbers and log the result.

```javascript
function add(a, b) {
    return a + b;
}

console.log(add(5, 7)); // 12
```

### 3.2 Parameters and Return Values

#### Explanation:
Functions can take parameters (inputs) and return a value. The `return` statement stops the execution of the function and returns a value.

#### Question 12: Parameters and Return Values
Write a function that takes an array as a parameter and returns the sum of its elements.

```javascript
function sumArray(arr) {
    let sum = 0;
    for (let i = 0; i < arr.length; i++) {
        sum += arr[i];
    }
    return sum;
}

console.log(sumArray([1, 2, 3, 4])); // 10
```

### 3.3 Arrow Functions

#### Explanation:
Arrow functions provide a shorter syntax for writing functions. They are always anonymous.

```javascript
// Traditional Function
function add(a, b) {
    return a + b;
}

// Arrow Function
const add = (a, b) => a + b;
```

#### Question 13: Arrow Functions
Rewrite the function from Question 12 using arrow function syntax.

```javascript
const sumArray = (arr) => {
    let sum = 0;
    for (let i = 0; i < arr.length; i++) {
        sum += arr[i];
    }
    return sum;
}

console.log(sumArray([1, 2, 3, 4])); // 10
```

This set of coding questions covers the basics of JavaScript, including variable declarations, data types, operators, control structures, and functions. These questions aim to help you understand and practice the fundamental concepts of JavaScript.