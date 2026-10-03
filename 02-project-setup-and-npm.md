## Table of Contents

* [Why Do We Need Project Setup?](#why-do-we-need-project-setup)
* [What is npm?](#what-is-npm)
* [Creating a New Project](#creating-a-new-project)
* [Initializing a Project](#initializing-a-project)
* [Understanding package.json](#understanding-packagejson)
* [npm init vs npm init -y](#npm-init-vs-npm-init--y)
* [Installing Packages](#installing-packages)
* [Uninstalling Packages](#uninstalling-packages)
* [What is node_modules?](#what-is-node_modules)
* [What is package-lock.json?](#what-is-package-lockjson)
* [Understanding Package Versions](#understanding-package-versions)
* [Sharing Your Project with Others](#sharing-your-project-with-others)
* [Dependencies vs Dev Dependencies](#dependencies-vs-dev-dependencies)
* [Understanding npm Scripts](#understanding-npm-scripts)
* [Installing Nodemon](#installing-nodemon)
* [Local vs Global Packages and npx](#local-vs-global-packages-and-npx)
* [Useful npm Commands](#useful-npm-commands)
* [Project Structure After Setup](#project-structure-after-setup)
* [Beginner Mistakes](#beginner-mistakes)
* [Practice Exercises](#practice-exercises)
* [Interview Questions](#interview-questions)

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

![Before npm vs After npm](images/02-project-setup-and-npm/before-after-npm.gif)

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
  "license": "ISC",
  "type": "commonjs"
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
| type        | Module system (commonjs means the project uses require). You will learn this in Session 04 |

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

### Installing Multiple Packages

Install several packages in one command by separating names with spaces

```bash
npm install express mongoose dotenv
```

All three are added to package.json together.

---

### Installing a Specific Version

By default npm installs the latest version.

Install a specific version using `@`

```bash
npm install express@4
```

```bash
npm install express@4.21.2
```

This is useful when a tutorial or project needs an older version.

---

### Short Commands

Developers often use shorter forms of the same commands.

| Full Command                   | Short Form              |
| ------------------------------ | ----------------------- |
| npm install express            | npm i express           |
| npm install nodemon --save-dev | npm i -D nodemon        |
| npm init --yes                 | npm init -y             |

Both forms do exactly the same thing.

---

## Uninstalling Packages

Remove a package you no longer need

```bash
npm uninstall express
```

npm will

* Delete the package from node_modules
* Remove it from package.json
* Update package-lock.json

Important

Always uninstall using npm.

Deleting a folder from node_modules manually leaves package.json unchanged, so the package comes back on the next install.

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

Important

Do not edit package-lock.json manually and do not delete it.

npm manages this file automatically.

---

## Understanding Package Versions

Every package version has three numbers.

![Semantic versioning: MAJOR.MINOR.PATCH](images/02-project-setup-and-npm/semantic-versioning.gif)

This system is called Semantic Versioning (SemVer).

Example

```text
5.1.0 -> 5.1.1    Bug fixed              Safe
5.1.0 -> 5.2.0    New feature added      Safe
5.1.0 -> 6.0.0    Breaking changes       Your code may need updates
```

---

### What does ^ mean?

In package.json you saw

```json
"express": "^5.1.0"
```

The `^` symbol means

```text
Allow MINOR and PATCH updates
Do not allow MAJOR updates
```

So `^5.1.0` can install `5.1.1` or `5.9.0`, but never `6.0.0`.

---

### Common Version Symbols

| Version | Meaning                     | Allowed Versions  |
| ------- | --------------------------- | ----------------- |
| ^5.1.0  | Minor and patch updates     | 5.1.0 to < 6.0.0  |
| ~5.1.0  | Patch updates only          | 5.1.0 to < 5.2.0  |
| 5.1.0   | Exact version only          | 5.1.0             |

npm uses `^` by default because it gives bug fixes and new features without breaking your code.

---

## Sharing Your Project with Others

node_modules can contain thousands of files and hundreds of megabytes.

It should never be shared or uploaded to GitHub.

Create a file named

```text
.gitignore
```

Add

```text
node_modules/
```

Git will now ignore the node_modules folder.

---

### What should be shared?

| File / Folder     | Share? | Reason                              |
| ----------------- | ------ | ----------------------------------- |
| package.json      | Yes    | Lists required packages and scripts |
| package-lock.json | Yes    | Locks exact versions                |
| Your code files   | Yes    | Your application                    |
| node_modules/     | No     | Can be recreated anytime            |

---

### Installing Dependencies of an Existing Project

When you download (clone) someone else's project, node_modules is missing.

Run

```bash
npm install
```

Without a package name, npm reads package.json and package-lock.json and installs everything the project needs.

Flow

![Clone, install and run a project](images/02-project-setup-and-npm/clone-install-flow.gif)

This is the first command you run in almost every Node.js project you download.

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

### npm start vs npm run

A few script names are special. They can run without the word `run`.

| Script Name | Command      |
| ----------- | ------------ |
| start       | npm start    |
| test        | npm test     |

Every other script needs `npm run`.

```json
{
  "scripts": {
    "start": "node index.js",
    "start:watch": "node --watch index.js",
    "dev": "nodemon index.js",
    "test": "node test.js"
  }
}
```

```bash
npm start
npm test
npm run dev
npm run start:watch
```

The `:` in `start:watch` is just part of the name. It is a common way to group related scripts.

In the Testing session you will see scripts like `test:watch` that follow the same idea.

---

### Listing All Scripts

To see every script available in a project

```bash
npm run
```

This is helpful when you open a new project and do not know which commands it uses.

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

Stop the application using

```text
Ctrl + C
```

---

### Nodemon vs node --watch

In Session 01 you learned

```bash
node --watch index.js
```

Both restart the application when files change.

| Feature              | node --watch       | nodemon                 |
| -------------------- | ------------------ | ----------------------- |
| Installation         | Built into Node.js | Must be installed       |
| Configuration        | Basic              | Many options            |
| Usage in projects    | Growing            | Very common             |

You will see nodemon in most existing projects and tutorials, so it is important to know both.

---

## Local vs Global Packages and npx

### Local Installation

```bash
npm install nodemon --save-dev
```

* Installed inside the project's node_modules
* Recorded in package.json
* Every developer gets the same version

This is the recommended way.

---

### Global Installation

```bash
npm install -g nodemon
```

* Installed once on your computer
* Usable from any folder
* Not recorded in package.json

Problem

Other developers will not know the project needs nodemon, and everyone may have a different version.

---

### Why Do Scripts Work Without Global Install?

When nodemon is installed locally, typing this in the terminal fails

```bash
nodemon index.js
```

```text
'nodemon' is not recognized as an internal or external command
```

But the same command works inside npm scripts.

```json
"dev": "nodemon index.js"
```

npm scripts automatically find tools inside the project's node_modules.

---

### What is npx?

npx runs a package without installing it globally.

```bash
npx nodemon index.js
```

npx will

* Use the local version if the package is installed in the project
* Otherwise download it temporarily and run it

Comparison

| Command | Purpose          |
| ------- | ---------------- |
| npm     | Install packages |
| npx     | Run packages     |

---

## Useful npm Commands

| Command                 | Purpose                                                   |
| ----------------------- | --------------------------------------------------------- |
| npm list                | Show installed packages                                   |
| npm outdated            | Show packages that have newer versions                    |
| npm update              | Update packages within allowed version range              |
| npm audit               | Check packages for known security problems                |
| npm audit fix           | Fix security problems automatically where possible        |
| npm ci                  | Clean install using exact versions from package-lock.json |
| npm cache clean --force | Clear npm's download cache (rarely needed)                |

Example

```bash
npm outdated
```

```text
Package  Current  Wanted  Latest
express  5.1.0    5.1.2   5.1.2
```

| Column  | Meaning                                  |
| ------- | ---------------------------------------- |
| Current | Installed version                        |
| Wanted  | Latest version allowed by package.json   |
| Latest  | Newest version available on npm          |

---

## Project Structure After Setup

Final project structure

```text
my-first-node-project/
|
├── node_modules/
├── .gitignore
├── package-lock.json
├── package.json
└── index.js
```

Understanding

![Project files and their roles](images/02-project-setup-and-npm/project-files-roles.gif)

---

## Beginner Mistakes

### Mistake 1

Running npm commands outside the project folder.

```bash
npm start
```

```text
npm error enoent Could not read package.json
```

Correct:

```bash
cd my-first-node-project
npm start
```

npm commands must run in the folder that contains package.json.

---

### Mistake 2

Incorrect:

```bash
npm dev
```

Correct:

```bash
npm run dev
```

Only `start` and `test` work without `run`.

---

### Mistake 3

Installing development tools as normal dependencies.

Incorrect:

```bash
npm install nodemon
```

Correct:

```bash
npm install nodemon --save-dev
```

---

### Mistake 4

Uploading node_modules to GitHub.

Correct:

Add `node_modules/` to `.gitignore` and share only package.json and package-lock.json.

---

### Mistake 5

Deleting package-lock.json to fix errors.

This can install different package versions and create new problems.

Correct:

```bash
npm ci
```

This reinstalls everything cleanly using the exact versions from package-lock.json.

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

### Exercise 5

Install `dotenv` and `cors` using a single command.

Then uninstall `cors`.

Verify that `cors` is removed from package.json.

---

### Exercise 6

Create a `.gitignore` file that ignores node_modules.

Delete the node_modules folder, then run

```bash
npm install
```

Verify that node_modules is created again with the same packages.

---

### Exercise 7

Run

```bash
npm outdated
npm run
```

Write down what each command shows.

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

### What does ^ mean in "express": "^5.1.0"?

It allows npm to install newer minor and patch versions (5.x.x) but not a new major version (6.0.0).

---

### Should node_modules be pushed to GitHub?

No. It is added to .gitignore. Other developers recreate it by running npm install.

---

### What is the difference between npm install and npm ci?

npm install can update package-lock.json while installing.

npm ci deletes node_modules and installs the exact versions from package-lock.json without changing it. It is commonly used on servers when deploying an application.

---

### What is the difference between npm and npx?

npm installs and manages packages.

npx runs a package without installing it globally.

---

### Why can we run npm start but not npm dev?

start and test are special script names npm recognizes directly. All other scripts need npm run.

---
