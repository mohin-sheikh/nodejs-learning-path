## Table of Contents

* [What is an Event?](#what-is-an-event)
* [Event Driven Programming](#event-driven-programming)
* [What is EventEmitter?](#what-is-eventemitter)
* [Creating Your First Event](#creating-your-first-event)
* [Listening to Events](#listening-to-events)
* [Passing Data with Events](#passing-data-with-events)
* [Multiple Event Listeners](#multiple-event-listeners)
* [Real World Example](#real-world-example)
* [Practice Exercises](#practice-exercises)
* [Interview Questions](#interview-questions)
* [Summary](#summary)

---

## What is an Event?

An event is an action or occurrence that happens in an application.

Examples

```text id="ft4ebj"
Button Clicked

User Registered

File Uploaded

Payment Completed

Order Created
```

Whenever something happens, an event can be triggered.

---

## Event Driven Programming

Node.js follows an event-driven architecture.

This means 

```text id="24d8ih"
Event Occurs -> Listener Detects Event -> Action Executes
```

Real-Life Example

```text id="1gnwaf"
Door Bell Rings -> Someone Opens The Door
```

The bell ringing is the event.

Opening the door is the action.

---

## What is EventEmitter?

Node.js provides a built-in class called 

```javascript id="ifsl6n"
EventEmitter
```

It helps us 

```text id="wz9b1g"
Create Events -> Listen To Events -> Trigger Events
```

Import EventEmitter

```javascript id="xvpfh6"
const EventEmitter = require("events");
```

Create an instance

```javascript id="0iwuxg"
const emitter = new EventEmitter();
```

---

## Creating Your First Event

Example

```javascript id="14hj2r"
const EventEmitter = require("events");

const emitter = new EventEmitter();

emitter.on("greet", () => {
  console.log("Hello Student");
});

emitter.emit("greet");
```

Output

```text id="n84mxz"
Hello Student
```

Explanation

```text id="49q12o"
on() -> Listen For Event

emit() -> Trigger Event
```

---

## Listening to Events

The `on()` method is used to listen for an event.

Example

```javascript id="2tfx6z"
emitter.on("login", () => {
  console.log("User Logged In");
});
```

Node.js waits until the event is triggered.

---

## Triggering Events

The `emit()` method is used to trigger an event.

Example

```javascript id="wfhfao"
emitter.emit("login");
```

Output

```text id="yjlwm8"
User Logged In
```

Flow

```text id="y6l8h7"
emit() -> Event Triggered

on() -> Code Executes
```

---

## Passing Data with Events

We can pass data when triggering an event.

Example

```javascript id="s3j4ga"
const EventEmitter = require("events");

const emitter = new EventEmitter();

emitter.on("welcome", (name) => {
  console.log(`Welcome ${name}`);
});

emitter.emit("welcome", "John");
```

Output

```text id="08lgz0"
Welcome John
```

Explanation

```text id="lmhpxw"
emit() -> Send Data

on() -> Receive Data
```

---

## Multiple Event Listeners

One event can have multiple listeners.

Example

```javascript id="6u7s9f"
const EventEmitter = require("events");

const emitter = new EventEmitter();

emitter.on("order", () => {
  console.log("Order Received");
});

emitter.on("order", () => {
  console.log("Payment Processing");
});

emitter.emit("order");
```

Output

```text id="js6o9t"
Order Received

Payment Processing
```

Flow

```text id="hjqtln"
Order Event -> Listener 1

Order Event -> Listener 2
```

---

## Real World Example

Imagine a user registration system.

When a user registers 

```text id="0v8gmb"
User Registered
        |
        v

Send Welcome Email

Create User Profile

Generate OTP

Write Logs
```

One event can trigger multiple actions.

Example

```javascript id="6gk9k5"
emitter.emit("userRegistered");
```

This makes applications easier to manage and scale.

---

## Event Flow Diagram

```text id="09f7tb"
Event Created
        |
        v

Listener Registered
        |
        v

Event Triggered
        |
        v

Listener Executes
```

---

## Practice Exercises

### Exercise 1

Create an event named 

```text id="ikowyk"
studentJoined
```

Print 

```text id="q4gd98"
New Student Joined
```

when the event is triggered.

---

### Exercise 2

Create an event named 

```text id="a3zyq0"
courseStarted
```

Pass the course name and display it.

Expected Output

```text id="owj0ep"
Course -> Node.js
```

---

### Exercise 3

Create two listeners for 

```text id="8i7jaf"
orderPlaced
```

Print different messages from each listener.

---

## Interview Questions

### What is an event?

An event is an action or occurrence that happens within an application.

---

### What is EventEmitter?

EventEmitter is a built-in Node.js class used to create and handle events.

---

### What does emit() do?

The emit() method triggers an event.

---

### What does on() do?

The on() method listens for an event.

---

### Can one event have multiple listeners?

Yes.

A single event can trigger multiple listeners.

---

## Summary

In this session, you learned

* What events are
* Event-driven programming
* EventEmitter
* Listening to events
* Triggering events
* Passing data with events
* Multiple event listeners

Events are one of the core concepts of Node.js and are used extensively in real-world applications.

---