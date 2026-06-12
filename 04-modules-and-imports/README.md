## Table of Contents

* [What is a Module?](#what-is-a-module)
* [Why Do We Need Modules?](#why-do-we-need-modules)
* [Creating Your First Module](#creating-your-first-module)
* [Using require()](#using-require)
* [Using module.exports](#using-moduleexports)
* [Exporting Multiple Functions](#exporting-multiple-functions)
* [Practice Exercises](#practice-exercises)
* [Interview Questions](#interview-questions)
* [Summary](#summary)

---

## What is a Module?

A module is simply a JavaScript file.

Example

```text
math.js
```

```text
user.js
```

```text
product.js
```

Each file can contain its own logic and functionality.

Think of modules as separate rooms in a house.

```text
House -> Bedroom

House -> Kitchen

House -> Living Room
```

Each room has a different purpose.

Similarly, each module should have a specific responsibility.

---

## Why Do We Need Modules?

Imagine writing an entire application inside one file.

```text
index.js -> 5000 Lines Of Code
```

Problems

* Difficult to read
* Difficult to debug
* Difficult to maintain

Better Approach

```text
project/

├── index.js
├── math.js
├── user.js
└── product.js
```

Each file handles a specific task.

---

## Creating Your First Module

Create a file

```text
math.js
```

Add 

```javascript
function add(a, b) {
  return a + b;
}

module.exports = add;
```

---

Create another file 

```text
index.js
```

Add 

```javascript
const add = require("./math");

console.log(add(10, 20));
```

Run 

```bash
node index.js
```

Output

```text
30
```

Congratulations. You have created and used your first custom module.

---

## Using require()

The require() function is used to import code from another file.

Example

```javascript
const add = require("./math");
```

Explanation

```text
require() -> Loads File -> Returns Exported Value
```

---

## Using module.exports

The module.exports object is used to share code with other files.

Example

```javascript
function add(a, b) {
  return a + b;
}

module.exports = add;
```

Explanation

```text
module.exports -> Makes Code Available -> For Other Files
```

Without module.exports, other files cannot access the function.

---

## Exporting Multiple Functions

math.js

```javascript
function add(a, b) {
  return a + b;
}

function subtract(a, b) {
  return a - b;
}

module.exports = {
  add,
  subtract
};
```

index.js

```javascript
const math = require("./math");

console.log(math.add(20, 10));

console.log(math.subtract(20, 10));
```

Output

```text
30

10
```

---

## Using Destructuring

Instead of

```javascript
const math = require("./math");

console.log(math.add(10, 5));
```

Use

```javascript
const { add } = require("./math");

console.log(add(10, 5));
```

Output

```text
15
```

This approach is commonly used in Node.js applications.

---

## Practice Exercises

### Exercise 1

Create 

```text
greeting.js
```

Export a function that returns 

```text
Hello Student
```

Import it inside 

```text
index.js
```

and print the result.

---

### Exercise 2

Create 

```text
calculator.js
```

Add 

* add()
* subtract()

functions.

Export them and use them inside 

```text
index.js
```

---

### Exercise 3

Create 

```text
student.js
```

Export 

```javascript
{
  name: "John",
  age: 20
}
```

Import and print the object.

---

## Interview Questions

### What is a module?

A module is a JavaScript file that contains reusable code.

---

### What is require()?

require() is used to import code from another module.

---

### What is module.exports?

module.exports is used to share code with other files.

---

### Why are modules important?

Modules help organize code and make applications easier to maintain.

---

## Summary

In this session, you learned

* What a module is
* Why modules are important
* How to create modules
* How to export code
* How to import code
* How to organize code into multiple files

These concepts are used in every Node.js application.

---