## Table of Contents

* [Why Learn How Node.js Works?](#why-learn-how-nodejs-works)
* [What Makes Node.js Different?](#what-makes-nodejs-different)
* [Single Threaded Architecture](#single-threaded-architecture)
* [Synchronous Programming](#synchronous-programming)
* [Asynchronous Programming](#asynchronous-programming)
* [Blocking Operations](#blocking-operations)
* [Non-Blocking Operations](#non-blocking-operations)
* [What is the Call Stack?](#what-is-the-call-stack)
* [What is the Event Loop?](#what-is-the-event-loop)
* [What is the Callback Queue?](#what-is-the-callback-queue)
* [How Event Loop Works](#how-event-loop-works)
* [Restaurant Example](#restaurant-example)
* [Node.js Architecture](#nodejs-architecture)
* [Why Node.js is Fast](#why-nodejs-is-fast)
* [Practice Questions](#practice-questions)
* [Interview Questions](#interview-questions)
* [Summary](#summary)
* [Next Session](#next-session)

---

## Why Learn How Node.js Works?

Many beginners learn Node.js like this

```javascript
app.get("/users", () => {});
```

or

```javascript
fs.readFile();
```

The code works.

But they do not understand what happens behind the scenes.

A good backend developer should understand

```text
Request -> Node.js -> Event Loop -> Operating System -> Response
```

This session focuses on understanding that process.

---

## What Makes Node.js Different?

Most backend technologies create multiple threads to handle multiple users.

Node.js uses a different approach.

Node.js uses

```text
Single Thread
    +
Event Loop
    +
Non-Blocking I/O
```

This combination allows Node.js to handle many requests efficiently.

---

## Single Threaded Architecture

One of the most common interview questions is

```text
Is Node.js Single Threaded?
```

Answer

```text
Yes
```

Node.js executes JavaScript code on a single main thread.

Many beginners think

```text
Single Thread -> Slow Application
```

This is incorrect.

Think about a restaurant.

Example

```text
One Cashier -> Handles Many Customers
```

The cashier does not cook food.

The cashier simply manages requests efficiently.

Node.js works similarly.

---

## Synchronous Programming

Synchronous code executes line by line.

Example

```javascript
console.log("Step 1");

console.log("Step 2");

console.log("Step 3");
```

Output

```text
Step 1
Step 2
Step 3
```

Execution Flow

```text
Step 1 -> Step 2 -> Step 3
```

Each operation waits for the previous operation to finish.

---

## Asynchronous Programming

Asynchronous code does not always wait.

Example

```javascript
console.log("Start");

setTimeout(() => {
  console.log("Timer Finished");
}, 2000);

console.log("End");
```

Output

```text
Start
End
Timer Finished
```

Many beginners expect

```text
Start
Timer Finished
End
```

But Node.js continues executing other code while the timer is running.

---

## Blocking Operations

Blocking means

```text
Wait Until Task Completes
```

Example

```text
Task 1 -> Wait -> Task 2
```

Real-Life Example

```text
Customer 1 -> Wait

Customer 2 -> Wait

Customer 3 -> Wait
```

Everyone waits in line.

---

## Non-Blocking Operations

Non-blocking means:

```text
Start Task -> Continue Other Work
```

Real-Life Example

```text
Customer 1 -> Order Taken

Customer 2 -> Order Taken

Customer 3 -> Order Taken
```

Nobody waits for cooking to finish before placing an order.

This is how Node.js handles many operations efficiently.

---

## What is the Call Stack?

The Call Stack keeps track of function execution.

Example

```javascript
function first() {
  second();
}

function second() {
  console.log("Hello");
}

first();
```

Call Stack Flow

```text
first() -> second() -> console.log()
```

Execution happens from top to bottom.

When a function completes, it is removed from the stack.

---

## What is the Event Loop?

The Event Loop is the heart of Node.js.

Its job is simple

```text
Check Call Stack -> Check Callback Queue -> Move Tasks For Execution
```

The Event Loop constantly runs in the background.

Think of it as a manager that decides what should execute next.

---

## What is the Callback Queue?

When asynchronous tasks finish, they do not directly enter the Call Stack.

Instead they move into the Callback Queue.

Example

```javascript
setTimeout(() => {
  console.log("Done");
}, 1000);
```

Flow

```text
Timer Starts -> Timer Finishes -> Callback Queue -> Event Loop -> Call Stack -> Execution
```

---

## How Event Loop Works

Consider

```javascript
console.log("Start");

setTimeout(() => {
  console.log("Timer");
}, 1000);

console.log("End");
```

Step 1

```text
Start
```

moves into Call Stack and executes.

Step 2

```text
setTimeout()
```

is registered.

Node.js starts the timer separately.

Step 3

```text
End
```

executes immediately.

Current Output

```text
Start
End
```

Step 4

Timer completes.

Callback enters Callback Queue.

Step 5

Event Loop notices Call Stack is empty.

Step 6

Callback moves into Call Stack.

Final Output

```text
Start
End
Timer
```

---

## Restaurant Example

This is one of the easiest ways to understand Node.js.

Restaurant

```text
Customer Places Order
        |
        v

Cashier Accepts Order
        |
        v

Kitchen Starts Cooking
        |
        v

Cashier Takes More Orders
        |
        v

Food Ready
        |
        v

Customer Receives Food
```

Node.js Mapping

```text
Cashier -> Event Loop

Kitchen -> Background Operations

Customer -> Request

Food -> Response
```

The cashier does not stop accepting orders while food is cooking.

Similarly, Node.js does not stop handling requests while waiting for operations to complete.

---

## Node.js Architecture

Simplified Architecture

```text
Application Code
        |
        v

Node.js Runtime
        |
        v

Event Loop
        |
        v

Operating System
```

The operating system handles many heavy operations.

Node.js manages communication between your application and the operating system.

---

## Why Node.js is Fast

Node.js is fast because it uses

```text
Single Thread
    +
Event Loop
    +
Non-Blocking I/O
```

Benefits

* Handles many requests efficiently
* Uses fewer resources
* Suitable for APIs
* Suitable for Real-Time Applications
* Suitable for Streaming Applications

---

## Practice Questions

### Question 1

What is the difference between synchronous and asynchronous code?

---

### Question 2

Why does Node.js use an Event Loop?

---

### Question 3

What is a Callback Queue?

---

### Question 4

Explain the restaurant example using Node.js concepts.

---

## Interview Questions

### Is Node.js Single Threaded?

Yes.

JavaScript execution happens on a single main thread.

---

### What is the Event Loop?

The Event Loop continuously checks the Call Stack and Callback Queue and manages task execution.

---

### What is Blocking Code?

Blocking code prevents the execution of other tasks until it finishes.

---

### What is Non-Blocking Code?

Non-blocking code allows other tasks to execute while waiting for an operation to complete.

---

### Why is Node.js Fast?

Because it uses

```text
Event Loop
    +
Non-Blocking I/O
```

which allows efficient handling of many requests.

---

## Summary

In this session, you learned

* Single Threaded Architecture
* Synchronous Programming
* Asynchronous Programming
* Blocking Operations
* Non-Blocking Operations
* Call Stack
* Callback Queue
* Event Loop
* Node.js Architecture
* Why Node.js is Fast

These concepts are the foundation of everything you will build in Node.js.

---