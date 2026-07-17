## Learning Objectives

After completing this session, you will be able to

* Create a Node.js project from scratch
* Understand the purpose of npm
* Initialize a project using npm
* Understand package.json
* Understand package-lock.json
* Install third-party packages
* Understand dependencies and devDependencies
* Create and run npm scripts
* Use nodemon during development

---

## Table of Contents

* [Why Do We Need Project Setup?](#why-do-we-need-project-setup)
* [What is npm?](#what-is-npm)
* [Creating a New Project](#creating-a-new-project)
* [Initializing a Project](#initializing-a-project)
* [Understanding package.json](#understanding-packagejson)
* [npm init vs npm init -y](#npm-init-vs-npm-init--y)
* [Installing Packages](#installing-packages)
* [What is node_modules?](#what-is-node_modules)
* [What is package-lock.json?](#what-is-package-lockjson)
* [Dependencies vs Dev Dependencies](#dependencies-vs-dev-dependencies)
* [Understanding npm Scripts](#understanding-npm-scripts)
* [Installing Nodemon](#installing-nodemon)
* [Project Structure After Setup](#project-structure-after-setup)
* [Practice Exercises](#practice-exercises)
* [Interview Questions](#interview-questions)
* [Summary](#summary)

---

## Why Do We Need Project Setup?

In the previous session, we executed JavaScript files directly using Node.js.

Example

```bash
node app.js
```

This approach works for learning and small examples.

However, real-world applications require

* Dependency management
* Version tracking
* Package installation
* Script execution
* Team collaboration
* Project metadata

Node.js projects solve these problems using npm.

Before npm

```text
JavaScript Files
      |
      v
Manual Management
```

After npm

```text
Project
      |
      v
package.json
      |
      v
Dependencies + Scripts + Metadata
```

---

## What is npm?

npm stands for

```text
Node Package Manager
```

npm is automatically installed when Node.js is installed.

npm helps developers

* Install libraries
* Manage dependencies
* Run project scripts
* Share packages
* Maintain project versions

Check npm version

```bash
npm -v
```

Example Output

```text
11.3.0
```

---

## Creating a New Project

Create a project folder

```bash
mkdir my-first-node-project
```

Move inside the folder

```bash
cd my-first-node-project
```

Current structure

```text
my-first-node-project/
```

The project is currently empty.

---

## Initializing a Project

To convert the folder into a Node.js project

```bash
npm init
```

npm will ask several questions.

Example

```text
package name:
version:
description:
entry point:
test command:
git repository:
keywords:
author:
license:
```

After answering these questions, npm creates

```text
package.json
```

This file is the heart of every Node.js project.

---

## Understanding package.json

Example

```json
{
  "name": "my-first-node-project",
  "version": "1.0.0",
  "description": "Learning Node.js",
  "main": "index.js",
  "scripts": {
    "start": "node index.js"
  },
  "author": "John Doe",
  "license": "ISC"
}
```

Explanation

| Property    | Description               |
| ----------- | ------------------------- |
| name        | Project name              |
| version     | Current project version   |
| description | Information about project |
| main        | Entry file                |
| scripts     | Custom commands           |
| author      | Project creator           |
| license     | Project license           |

Think of package.json as

```text
Project Identity Card
```

It contains everything npm needs to know about the project.

---

## npm init vs npm init -y

### Interactive Initialization

```bash
npm init
```

npm asks questions one by one.

---

### Quick Initialization

```bash
npm init -y
```

Creates package.json automatically.

Most developers use:

```bash
npm init -y
```

because it is faster.

Generated structure:

```text
my-first-node-project/
|
└── package.json
```

---

## Installing Packages

Node.js becomes powerful because of packages.

Install Express

```bash
npm install express
```

After installation:

```text
my-first-node-project/
|
├── node_modules/
├── package-lock.json
└── package.json
```

package.json is updated automatically:

```json
{
  "dependencies": {
    "express": "^5.1.0"
  }
}
```

---

## What is node_modules?

After installing packages, npm creates:

```text
node_modules/
```

This folder contains

* Installed packages
* Package dependencies
* Internal package files

Example

```text
node_modules/
├── express
├── body-parser
├── cookie
├── debug
└── many more packages
```

Important

Do not modify files inside node_modules.

Think of node_modules as

```text
Downloaded Libraries Storage
```

---

## What is package-lock.json?

Beginners often confuse package.json and package-lock.json.

### package.json

Stores

```text
Required Packages
```

Example

```json
{
  "dependencies": {
    "express": "^5.1.0"
  }
}
```

---

### package-lock.json

Stores

```text
Exact Installed Versions
```

Example

```text
express -> 5.1.0
dependency-a -> 2.0.4
dependency-b -> 1.3.7
```

This ensures all developers install the same package versions.

---

## Dependencies vs Dev Dependencies

### Dependency

Required when application runs.

Install

```bash
npm install express
```

Stored under:

```json
{
  "dependencies": {}
}
```

Examples:

* Express
* Mongoose
* JWT

---

### Dev Dependency

Required only during development.

Install

```bash
npm install nodemon --save-dev
```

Stored under

```json
{
  "devDependencies": {}
}
```

Examples

* Nodemon
* ESLint
* Jest

---

## Understanding npm Scripts

Without scripts

```bash
node index.js
```

every time.

With scripts

```json
{
  "scripts": {
    "start": "node index.js"
  }
}
```

Run

```bash
npm start
```

This is easier and more professional.

---

## Installing Nodemon

Create

```text
index.js
```

```javascript
console.log("Node.js Project Started");
```

Install nodemon

```bash
npm install nodemon --save-dev
```

Update scripts

```json
{
  "scripts": {
    "dev": "nodemon index.js"
  }
}
```

Run

```bash
npm run dev
```

Now every file save automatically restarts the application.

---

## Project Structure After Setup

Final project structure

```text
my-first-node-project/
|
├── node_modules/
├── package-lock.json
├── package.json
└── index.js
```

Understanding

```text
index.js
    |
    -> Application Entry Point

package.json
    |
    -> Project Configuration

package-lock.json
    |
    -> Exact Package Versions

node_modules
    |
    -> Installed Libraries
```

---

## Practice Exercises

### Exercise 1

Create a project named

```text
student-management
```

Initialize npm using

```bash
npm init -y
```

---

### Exercise 2

Create

```text
index.js
```

Print

```javascript
console.log("Student Management System");
```

---

### Exercise 3

Install Express.

Verify:

```text
node_modules
package-lock.json
```

are created.

---

### Exercise 4

Install Nodemon.

Create a script:

```json
"dev": "nodemon index.js"
```

Run:

```bash
npm run dev
```

---

## Interview Questions

### What is npm?

npm stands for Node Package Manager and is used for package management.

---

### What is package.json?

package.json contains project metadata, scripts, and dependency information.

---

### What is node_modules?

node_modules stores installed packages and their dependencies.

---

### What is package-lock.json?

package-lock.json stores exact package versions.

---

### What is the difference between dependency and devDependency?

Dependencies are required in production.

DevDependencies are required only during development.

---

### What does npm init -y do?

It creates package.json with default values.

---

## Summary

In this session, you learned

* Why project setup is important
* What npm is
* How to initialize a Node.js project
* How package.json works
* How package-lock.json works
* How to install packages
* What node_modules contains
* How npm scripts work
* How to use nodemon

---
