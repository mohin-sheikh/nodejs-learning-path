## Table of Contents

* [What is the Path Module?](#what-is-the-path-module)
* [Importing the Path Module](#importing-the-path-module)
* [Why Do We Need the Path Module?](#why-do-we-need-the-path-module)
* [path.join()](#pathjoin)
* [path.basename()](#pathbasename)
* [path.extname()](#pathextname)
* [path.dirname()](#pathdirname)
* [path.parse()](#pathparse)
* [Practice Exercises](#practice-exercises)
* [Interview Questions](#interview-questions)
* [Summary](#summary)

---

## What is the Path Module?

The Path Module helps us work with file and folder paths.

It is a built-in Node.js module.

No installation is required.

The Path Module helps us 

```text
- Create Paths
- Get File Names
- Get File Extensions
- Get Folder Paths
```

---

## Importing the Path Module

```javascript
const path = require("path");
```

Explanation

```text
path -> Built-in Node.js Module
```

---

## Why Do We Need the Path Module?

Imagine we have a file 

```text
project/
├── data/
│   └── student.txt
└── app.js
```

Without the Path Module 

```javascript
const filePath = "data/student.txt";
```

This may behave differently across operating systems.

With the Path Module 

```javascript
const filePath = path.join(
  "data",
  "student.txt"
);
```

Node.js automatically creates the correct path.

---

## path.join()

Used to join multiple path segments.

Example

```javascript
const path = require("path");

const filePath = path.join(
  "data",
  "student.txt"
);

console.log(filePath);
```

Output

```text
data/student.txt
```

Purpose

```text
Join Multiple Path Segments
```

This is one of the most commonly used path methods.

---

## path.basename()

Used to get the file name.

Example

```javascript
const path = require("path");

const fileName = path.basename(
  "/data/student.txt"
);

console.log(fileName);
```

Output

```text
student.txt
```

Purpose

```text
Get File Name
```

---

## path.extname()

Used to get the file extension.

Example

```javascript
const path = require("path");

const extension = path.extname(
  "student.txt"
);

console.log(extension);
```

Output

```text
.txt
```

Purpose

```text
Get File Extension
```

---

## path.dirname()

Used to get the directory path.

Example

```javascript
const path = require("path");

const directory = path.dirname(
  "/data/student.txt"
);

console.log(directory);
```

Output

```text
/data
```

Purpose

```text
Get Folder Path
```

---

## path.parse()

Used to get detailed information about a file path.

Example

```javascript
const path = require("path");

const fileInfo = path.parse(
  "student.txt"
);

console.log(fileInfo);
```

Output

```javascript
{
  root: "",
  dir: "",
  base: "student.txt",
  ext: ".txt",
  name: "student"
}
```

Purpose

```text
Get Complete Path Information
```

---

## Real World Example

Project Structure

```text
project/
├── data/
│   └── students.json
└── app.js
```

app.js

```javascript
const path = require("path");

const filePath = path.join(
  "data",
  "students.json"
);

console.log(filePath);
```

Output

```text
data/students.json
```

This is how paths are commonly created in real applications.

---

## Practice Exercises

### Exercise 1

Create a path for 

```text
data -> students.json
```

using 

```javascript
path.join()
```

---

### Exercise 2

Get the file name from 

```text
/uploads/profile.jpg
```

---

### Exercise 3

Get the extension from 

```text
resume.pdf
```

---

### Exercise 4

Get the directory path from 

```text
/data/students/student.txt
```

---

## Interview Questions

### What is the Path Module?

The Path Module is a built-in Node.js module used to work with file and folder paths.

---

### Do we need to install the Path Module?

No.

It comes bundled with Node.js.

---

### What does path.join() do?

It combines multiple path segments into a single path.

---

### What does path.basename() return?

It returns the file name.

---

### What does path.extname() return?

It returns the file extension.

---

## Summary

In this session, you learned 

* What the Path Module is
* How to create paths
* How to get file names
* How to get file extensions
* How to get directory names
* How to extract path information

The Path Module is commonly used together with the File System Module.

---