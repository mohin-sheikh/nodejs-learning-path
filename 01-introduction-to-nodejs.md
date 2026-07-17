# Introduction to Node.js

Welcome to the first session of the Node.js learning journey.

In this session, we will understand what Node.js is, why it was created, where it is used, and how to set it up on our system. By the end of this session, you will be able to write and execute your first Node.js program.

---
## Table of Contents

* [Learning Objectives](#learning-objectives)
* [What is Node.js?](#what-is-nodejs)
* [Why Node.js?](#why-nodejs)
* [Where is Node.js Used?](#where-is-nodejs-used)
* [Companies Using Node.js](#companies-using-nodejs)
* [Installing Node.js](#installing-nodejs)
* [Verifying Installation](#verifying-installation)
* [Understanding npm](#understanding-npm)
* [Creating Your First Node.js Program](#creating-your-first-nodejs-program)
* [Running a Node.js Program](#running-a-nodejs-program)
* [Understanding the Command](#understanding-the-command)
* [More Examples](#more-examples)
* [Common Node.js Commands](#common-nodejs-commands)
* [Practice Exercises](#practice-exercises)
* [Beginner Mistakes](#beginner-mistakes)
* [Interview Questions](#interview-questions)
* [Summary](#summary)

---


# Learning Objectives

After completing this session, you will be able to

* Understand what Node.js is
* Understand why Node.js was created
* Identify common use cases of Node.js
* Install Node.js on your system
* Verify the installation
* Understand the role of npm
* Create and run your first Node.js application
* Use basic Node.js commands

---

# What is Node.js?

Node.js is an open-source JavaScript runtime environment that allows JavaScript code to run outside the browser.

Before Node.js, JavaScript could only be executed inside web browsers such as

* Google Chrome
* Mozilla Firefox
* Microsoft Edge
* Safari

Node.js changed this by allowing developers to run JavaScript directly on a computer or server.

Simply put

> Node.js allows JavaScript to be used for backend development.

---

# Why Node.js?

Before Node.js, developers commonly used

* PHP
* Java
* C#
* Python

for backend development.

Node.js introduced the ability to use JavaScript for both

### Frontend

```javascript
console.log("Running in Browser");
```

### Backend

```javascript
console.log("Running on Server");
```

This means developers can use a single programming language across the entire application.

Benefits:

* Easy to learn for JavaScript developers
* Fast execution using the V8 Engine
* Large ecosystem of packages
* Great community support
* Ideal for APIs and real-time applications

---

# Where is Node.js Used?

Node.js is commonly used for

### REST APIs

Building backend services for web and mobile applications.

Examples -

* E-commerce Applications
* Food Delivery Applications
* Social Media Applications

### Real-Time Applications

Applications requiring instant communication.

Examples -

* Chat Applications
* Live Notifications
* Online Gaming

### Microservices

Large applications split into smaller independent services.

### Streaming Applications

Examples -

* Video Streaming
* Music Streaming

---

# Companies Using Node.js

Some well-known companies using Node.js include

* Netflix
* PayPal
* Uber
* LinkedIn
* Walmart

---

# Installing Node.js

## Step 1

Visit the official Node.js website 

https://nodejs.org

## Step 2

Download the LTS (Long Term Support) version.

LTS is recommended because it is stable and widely used in production environments.

## Step 3

Install Node.js using the default installation options.

---

# Verifying Installation

Open a terminal and run

```bash
node -v
```

Example Output:

```bash
v24.2.0
```

Check npm version:

```bash
npm -v
```

Example Output:

```bash
11.3.0
```

If both commands return version numbers, Node.js has been installed successfully.

---

# Understanding npm

npm stands for

```text
Node Package Manager
```

npm is automatically installed along with Node.js.

It is used to

* Install packages
* Manage project dependencies
* Execute scripts
* Share libraries

Examples:

Install Express.js

```bash
npm install express
```

Install Nodemon:

```bash
npm install nodemon
```

Install Mongoose:

```bash
npm install mongoose
```

We will learn these packages in future sessions.

---

# Creating Your First Node.js Program

Create a folder

```text
nodejs-learning
```

Inside the folder create

```text
app.js
```

Add the following code

```javascript
console.log("Hello Node.js");
```

Folder structure

```text
nodejs-learning
└── app.js
```

---

# Running a Node.js Program

Open terminal inside the project folder.

Run:

```bash
node app.js
```

Output:

```bash
Hello Node.js
```

You have successfully executed your first Node.js application.

---

# Understanding the Command

Command

```bash
node app.js
```

Explanation:

| Part   | Description                |
| ------ | -------------------------- |
| node   | Starts the Node.js runtime |
| app.js | File to execute            |

---

# More Examples

## Example 1

```javascript
console.log("Welcome to Backend Development");
```

Output:

```bash
Welcome to Backend Development
```

---

## Example 2

```javascript
const name = "Mohin";

console.log(name);
```

Output:

```bash
Mohin
```

---

## Example 3

```javascript
const num1 = 10;
const num2 = 20;

console.log(num1 + num2);
```

Output:

```bash
30
```

---

# Common Node.js Commands

## Check Node Version

```bash
node -v
```

---

## Check npm Version

```bash
npm -v
```

---

## Execute a File

```bash
node app.js
```

---

## Watch File Changes

```bash
node --watch app.js
```

Node.js automatically restarts whenever the file changes.

---

## Display Help Information

```bash
node --help
```

---

# Practice Exercises

## Exercise 1

Create a file named:

```text
welcome.js
```

Print:

```text
Welcome to Node.js
```

---

## Exercise 2

Create two variables and print their sum.

Expected Output:

```text
75
```

---

## Exercise 3

Print your:

* Name
* Age
* City

using console.log()

---

## Exercise 4

Create a file named:

```text
student.js
```

Store student information inside variables and print them.

---

# Beginner Mistakes

### Mistake 1

Incorrect:

```bash
node app
```

Correct:

```bash
node app.js
```

---

### Mistake 2

Running commands outside the project folder.

Always navigate to the project folder first.

---

### Mistake 3

Typing:

```bash
Node app.js
```

Correct:

```bash
node app.js
```

Node commands are lowercase.

---

# Interview Questions

### What is Node.js?

Node.js is a JavaScript runtime environment that allows JavaScript to run outside the browser.

---

### Is Node.js a Programming Language?

No.

Node.js is a runtime environment.

---

### What is npm?

npm stands for Node Package Manager.

It is used for managing packages and dependencies.

---

### Can JavaScript run without a browser?

Yes.

Using Node.js, JavaScript can run outside the browser.

---

# Summary

In this session, you learned

* What Node.js is
* Why Node.js is used
* Where Node.js is used
* How to install Node.js
* How to verify installation
* What npm is
* How to create your first Node.js program
* Common Node.js commands

---