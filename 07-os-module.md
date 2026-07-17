## Table of Contents

* [What is the OS Module?](#what-is-the-os-module)
* [Importing the OS Module](#importing-the-os-module)
* [Getting Platform Information](#getting-platform-information)
* [Getting System Architecture](#getting-system-architecture)
* [Getting Host Name](#getting-host-name)
* [Getting Memory Information](#getting-memory-information)
* [Getting CPU Information](#getting-cpu-information)
* [Practice Exercises](#practice-exercises)
* [Interview Questions](#interview-questions)
* [Summary](#summary)

---

## What is the OS Module?

The OS Module provides information about the operating system on which Node.js is running.

Using this module, we can get 

```text
- Operating System
- Architecture
- Memory Information
- CPU Information
```

The OS Module is built into Node.js.

No installation is required.

---

## Importing the OS Module

```javascript
const os = require("os");
```

Explanation

```text
os -> Operating System Module
```

---

## Getting Platform Information

The `platform()` method returns the operating system platform.

Example

```javascript
const os = require("os");

console.log(os.platform());
```

Possible Output

```text
win32
```

```text
linux
```

```text
darwin
```

Common Platforms

| Platform | Operating System |
| -------- | ---------------- |
| win32    | Windows          |
| linux    | Linux            |
| darwin   | macOS            |

---

## Getting System Architecture

The `arch()` method returns the system architecture.

Example

```javascript
const os = require("os");

console.log(os.arch());
```

Output

```text
x64
```

Explanation

```text
x64 -> 64 Bit System
```

---

## Getting Host Name

The `hostname()` method returns the computer name.

Example

```javascript
const os = require("os");

console.log(os.hostname());
```

Output

```text
DESKTOP-ABC123
```

Your output will be different.

---

## Getting Memory Information

### Total Memory

```javascript
const os = require("os");

console.log(os.totalmem());
```

Output

```text
17071734784
```

The value is returned in bytes.

---

### Free Memory

```javascript
const os = require("os");

console.log(os.freemem());
```

Output

```text
8423454720
```

The value is returned in bytes.

Explanation

```text
totalmem() -> Total RAM

freemem() -> Available RAM
```

---

## Getting CPU Information

The `cpus()` method returns information about available CPUs.

Example

```javascript
const os = require("os");

console.log(os.cpus());
```

Output

```text
[
  {
    model: 'Intel...',
    speed: 2500
  }
]
```

This method returns detailed CPU information.

---

## Mini Example

Create 

```text
system-info.js
```

```javascript
const os = require("os");

console.log("Platform ->", os.platform());

console.log("Architecture ->", os.arch());

console.log("Host Name ->", os.hostname());
```

Run

```bash
node system-info.js
```

Output

```text
Platform -> win32

Architecture -> x64

Host Name -> DESKTOP-ABC123
```

Your output may differ.

---

## Practice Exercises

### Exercise 1

Display the operating system platform.

Expected Method

```javascript
os.platform()
```

---

### Exercise 2

Display the system architecture.

Expected Method

```javascript
os.arch()
```

---

### Exercise 3

Display the host name of your computer.

Expected Method

```javascript
os.hostname()
```

---

### Exercise 4

Display 

```text
Total Memory

Free Memory
```

using the OS Module.

---

## Interview Questions

### What is the OS Module?

The OS Module is a built-in Node.js module used to get operating system information.

---

### Do we need to install the OS Module?

No.

It is included with Node.js.

---

### What does os.platform() return?

It returns the operating system platform.

Examples 

```text
win32

linux

darwin
```

---

### What does os.arch() return?

It returns the system architecture.

Example 

```text
x64
```

---

### What does os.hostname() return?

It returns the name of the computer.

---

## Summary

In this session, you learned 

* What the OS Module is
* How to get platform information
* How to get architecture information
* How to get host name information
* How to get memory information
* How to get CPU information

The OS Module helps applications understand the environment in which they are running.

---