## Table of Contents

* [What is the File System Module?](#what-is-the-file-system-module)
* [Importing fs Module](#importing-fs-module)
* [Reading a File](#reading-a-file)
* [Creating a File](#creating-a-file)
* [Updating a File](#updating-a-file)
* [Deleting a File](#deleting-a-file)
* [Sync vs Async Methods](#sync-vs-async-methods)
* [Practice Exercises](#practice-exercises)
* [Interview Questions](#interview-questions)
* [Summary](#summary)
* [Next Session](#next-session)

---

## What is the File System Module?

The File System Module allows Node.js to interact with files stored on your computer.

Using the fs module, we can

```text
- Create Files
- Read Files
- Update Files
- Delete Files
```

The fs module is a built-in Node.js module.

No installation is required.

---

## Importing fs Module

Before using the File System Module, import it 

```javascript
const fs = require("fs");
```

Explanation

```text
fs -> File System Module
```

---

## Reading a File

Create a file 

```text
student.txt
```

Content 

```text
Welcome To Node.js
```

Read the file 

```javascript
const fs = require("fs");

const data = fs.readFileSync("student.txt", "utf8");

console.log(data);
```

Output

```text
Welcome To Node.js
```

---

## Creating a File

Create a new file 

```javascript
const fs = require("fs");

fs.writeFileSync(
  "course.txt",
  "Node.js Learning Path"
);
```

Result

```text
course.txt
```

is created automatically.

---

## Updating a File

Add content to an existing file 

```javascript
const fs = require("fs");

fs.appendFileSync(
  "course.txt",
  "\nSession 05 -> File System Module"
);
```

Updated Content

```text
Node.js Learning Path
Session 05 -> File System Module
```

---

## Deleting a File

Delete a file 

```javascript
const fs = require("fs");

fs.unlinkSync("course.txt");
```

Result

```text
course.txt  -> Deleted
```

---

## Sync vs Async Methods

Node.js provides two types of methods.

### Synchronous Method

```javascript
fs.readFileSync();
```

Behavior

```text
Start -> Wait For Completion -> Continue
```

---

### Asynchronous Method

```javascript
fs.readFile(
  "student.txt",
  "utf8",
  (err, data) => {
    console.log(data);
  }
);
```

Behavior

```text
Start Task -> Continue Other Work -> Execute Callback
```

Async methods are generally preferred in real-world applications.

---

## Mini Example

Create 

```text
notes.txt
```

Write Data

```javascript
fs.writeFileSync(
  "notes.txt",
  "Learning Node.js"
);
```

Read Data

```javascript
const data = fs.readFileSync(
  "notes.txt",
  "utf8"
);

console.log(data);
```

Output

```text
Learning Node.js
```

---

## Practice Exercises

### Exercise 1

Create 

```text
student.txt
```

Store 

```text
My Name Is John
```

using Node.js.

---

### Exercise 2

Read and display the contents of 

```text
student.txt
```

---

### Exercise 3

Append 

```text
I Am Learning Node.js
```

to the same file.

---

### Exercise 4

Delete 

```text
student.txt
```

using Node.js.

---

## Interview Questions

### What is the fs module?

The fs module is a built-in Node.js module used to work with files and folders.

---

### Do we need to install fs?

No.

It comes bundled with Node.js.

---

### What is the difference between readFileSync() and readFile()?

```text
readFileSync() -> Synchronous

readFile() -> Asynchronous
```

---

### Which is preferred in production?

Asynchronous methods are generally preferred because they do not block the event loop.

---

## Summary

In this session, you learned

* What the fs module is
* How to read files
* How to create files
* How to update files
* How to delete files
* Difference between Sync and Async methods

The File System Module is one of the most commonly used built-in modules in Node.js.

---