## Table of Contents

* [What are Environment Variables](#what-are-environment-variables)
* [Why Do We Need Environment Variables](#why-do-we-need-environment-variables)
* [What is dotenv](#what-is-dotenv)
* [Installing dotenv](#installing-dotenv)
* [Creating .env File](#creating-env-file)
* [Loading Environment Variables](#loading-environment-variables)
* [Accessing Environment Variables](#accessing-environment-variables)
* [Using process.env](#using-processenv)
* [Default Values](#default-values)
* [Environment Variables in Express](#environment-variables-in-express)
* [Different Environments](#different-environments)
* [Important Rules for .env](#important-rules-for-env)
* [Complete Example](#complete-example)
* [Practice Exercises](#practice-exercises)
* [Interview Questions](#interview-questions)
* [Summary](#summary)

---

## What are Environment Variables

Environment variables are values that live outside your code

They are stored in the system where your application runs

Think of them like settings for your application

Examples of environment variables

```text
PORT -> Which port the server runs on
DATABASE_URL -> Where your database is located
API_KEY -> Secret key for external services
NODE_ENV -> Current environment (development, production)
```

Environment variables are key-value pairs

```text
KEY = VALUE
PORT = 3000
DATABASE_URL = mongodb://localhost:27017/myapp
API_KEY = abc123secret
```

---

## Why Do We Need Environment Variables

Hardcoding values in your code is a bad practice

Bad example - hardcoded values

```javascript
const PORT = 3000;
const DATABASE_URL = "mongodb://localhost:27017/school";
const API_KEY = "my-secret-key-123";
```

Problems with hardcoding

```text
Cannot change without editing code
Different developers need different settings
Secret keys are visible in code
Cannot have different settings for different computers
Cannot share code safely
```

Good example - using environment variables

```javascript
const PORT = process.env.PORT;
const DATABASE_URL = process.env.DATABASE_URL;
const API_KEY = process.env.API_KEY;
```

Benefits

```text
No hardcoded values
Each developer can have their own .env file
Secret keys are not in the code
Easy to change settings
Safe to share code on GitHub
```

---

## What is dotenv

dotenv is a npm package

It loads environment variables from a .env file into process.env

Without dotenv, you would have to set environment variables manually

Manually setting on Windows

```bash
set PORT=5000
node server.js
```

Manually setting on Mac/Linux

```bash
export PORT=5000
node server.js
```

This is annoying and easy to forget

dotenv makes it simple

Create a .env file with your variables

```text
PORT=5000
DATABASE_URL=mongodb://localhost:27017/mydb
```

dotenv loads them automatically

---

## Installing dotenv

Create a new project

```bash
mkdir env-demo
cd env-demo
npm init -y
```

Install express

```bash
npm install express
```

Install dotenv

```bash
npm install dotenv
```

Now you have dotenv in your project

---

## Creating .env File

Create a file named .env in your project root

Important - The name must be exactly .env

Add your variables

```text
PORT=3000
NODE_ENV=development
API_KEY=my-super-secret-key-123
DATABASE_URL=mongodb://localhost:27017/school
ADMIN_EMAIL=admin@example.com
```

Each line is one variable

No spaces around the equals sign

No quotes around values

---

## Loading Environment Variables

There are two ways to load dotenv

Method 1 - Require and configure at the top of your file

```javascript
require("dotenv").config();

const express = require("express");
const app = express();

console.log(process.env.PORT);
```

Method 2 - Import if using ES modules

```javascript
import "dotenv/config";
```

For beginners, Method 1 is easier

The config() function reads your .env file and loads variables

Always put dotenv at the very top of your file

---

## Accessing Environment Variables

Once loaded, all variables are available in process.env

```javascript
require("dotenv").config();

console.log(process.env.PORT);
console.log(process.env.NODE_ENV);
console.log(process.env.API_KEY);
console.log(process.env.DATABASE_URL);
console.log(process.env.ADMIN_EMAIL);
```

Output

```text
3000
development
my-super-secret-key-123
mongodb://localhost:27017/school
admin@example.com
```

process.env is an object

All environment variables become properties of this object

---

## Using process.env

Access variables using dot notation

```javascript
const port = process.env.PORT;
const dbUrl = process.env.DATABASE_URL;
```

Or using bracket notation

```javascript
const port = process.env["PORT"];
```

Most developers use dot notation

Now you can use these variables in your code

```javascript
const express = require("express");
require("dotenv").config();

const app = express();
const PORT = process.env.PORT;

app.get("/", (req, res) => {
  res.send("Server using environment variables");
});

app.listen(PORT, () => {
  console.log(`Server running on port ${PORT}`);
});
```

---

## Default Values

Sometimes an environment variable might not exist

You should provide a default value

Use the OR operator ||

```javascript
const PORT = process.env.PORT || 3000;
const DB_URL = process.env.DATABASE_URL || "mongodb://localhost:27017/default";
```

If process.env.PORT exists, it uses that value

If not, it uses 3000

This is good practice

Your app will still work even if someone forgets to set a variable

---

## Environment Variables in Express

Here is a complete Express server using environment variables

```javascript
require("dotenv").config();
const express = require("express");

const app = express();

const PORT = process.env.PORT || 5000;
const NODE_ENV = process.env.NODE_ENV || "development";

app.use(express.json());

app.get("/", (req, res) => {
  res.json({
    message: "Server is running",
    environment: NODE_ENV,
    port: PORT
  });
});

app.get("/config", (req, res) => {
  // Never send secret keys to client
  // This is just for demonstration
  res.json({
    nodeEnv: NODE_ENV,
    // Do NOT send API keys to client in real apps
  });
});

app.listen(PORT, () => {
  console.log(`Server running in ${NODE_ENV} mode on port ${PORT}`);
});
```

---

## Different Environments

Most applications have different environments

```text
development -> For coding and testing on your computer
staging -> For testing before going live
production -> Live application used by real users
```

Each environment has different settings

.env.development

```text
PORT=3000
DATABASE_URL=mongodb://localhost:27017/dev_db
DEBUG=true
```

.env.production

```text
PORT=8080
DATABASE_URL=mongodb://prod-server:27017/prod_db
DEBUG=false
```

You can create different .env files for different environments

Then load the correct one based on NODE_ENV

```javascript
const env = process.env.NODE_ENV || "development";

if (env === "development") {
  require("dotenv").config({ path: ".env.development" });
} else if (env === "production") {
  require("dotenv").config({ path: ".env.production" });
} else {
  require("dotenv").config();
}
```

For beginners, start with just one .env file

---

## Important Rules for .env

Rule 1 - Never commit .env to GitHub

Add .env to .gitignore

```text
# .gitignore
node_modules/
.env
.env.*
```

Rule 2 - Create a .env.example file

Share this file with your team

.env.example

```text
PORT=3000
NODE_ENV=development
DATABASE_URL=mongodb://localhost:27017/mydb
API_KEY=your-api-key-here
```

Team members copy .env.example to .env

Then fill in their own values

Rule 3 - No spaces around equals

```text
Good -> PORT=3000
Bad  -> PORT = 3000
```

Rule 4 - No quotes around values

```text
Good -> API_KEY=abc123
Bad  -> API_KEY="abc123"
```

Rule 5 - Restart server after changing .env

Changes to .env only take effect when you restart the server

---

## Complete Example

Project structure

```text
env-demo/
├── .env
├── .env.example
├── .gitignore
├── package.json
└── server.js
```

.env file

```text
PORT=4000
NODE_ENV=development
JWT_SECRET=my-jwt-secret-key
EMAIL_HOST=smtp.gmail.com
EMAIL_PORT=587
EMAIL_USER=admin@myapp.com
EMAIL_PASS=email-password-here
RATE_LIMIT=100
```

.env.example file (share this on GitHub)

```text
PORT=3000
NODE_ENV=development
JWT_SECRET=your-jwt-secret-key
EMAIL_HOST=smtp.gmail.com
EMAIL_PORT=587
EMAIL_USER=your-email@example.com
EMAIL_PASS=your-email-password
RATE_LIMIT=100
```

.gitignore file

```text
node_modules/
.env
.env.*
.DS_Store
```

server.js

```javascript
require("dotenv").config();
const express = require("express");

const app = express();

// Get all configuration from environment variables
const PORT = process.env.PORT || 3000;
const NODE_ENV = process.env.NODE_ENV || "development";
const JWT_SECRET = process.env.JWT_SECRET;
const RATE_LIMIT = process.env.RATE_LIMIT || 100;

// Check if required variables exist
if (!JWT_SECRET) {
  console.error("FATAL ERROR: JWT_SECRET is not defined");
  process.exit(1);
}

app.use(express.json());

// Route to show config (without secrets)
app.get("/api/config", (req, res) => {
  res.json({
    environment: NODE_ENV,
    port: PORT,
    rateLimit: RATE_LIMIT,
    // Never send JWT_SECRET or passwords to client
  });
});

app.get("/api/health", (req, res) => {
  res.json({
    status: "OK",
    environment: NODE_ENV,
    timestamp: new Date().toISOString()
  });
});

app.listen(PORT, () => {
  console.log(`=================================`);
  console.log(`Server started successfully`);
  console.log(`=================================`);
  console.log(`Environment: ${NODE_ENV}`);
  console.log(`Port: ${PORT}`);
  console.log(`Rate limit: ${RATE_LIMIT} requests per minute`);
  console.log(`=================================`);
  console.log(`Visit http://localhost:${PORT}/api/health`);
});
```

Run the server

```bash
node server.js
```

Output

```text
=================================
Server started successfully
=================================
Environment: development
Port: 4000
Rate limit: 100 requests per minute
=================================
Visit http://localhost:4000/api/health
```

---

## Environment Variables in Different Operating Systems

Setting environment variables without .env

Windows Command Prompt

```bash
set PORT=5000
node server.js
```

Windows PowerShell

```bash
$env:PORT=5000
node server.js
```

Mac or Linux

```bash
export PORT=5000
node server.js
```

This is why dotenv is better

Works the same on all operating systems

---

## Common Environment Variables

Here are variables most projects use

```text
PORT -> Server port number
NODE_ENV -> development, staging, production
DATABASE_URL -> Database connection string
JWT_SECRET -> Secret key for JWT tokens
API_KEY -> External API keys
CORS_ORIGIN -> Allowed domains for CORS
LOG_LEVEL -> debug, info, warn, error
SESSION_SECRET -> Secret for session management
EMAIL_HOST -> SMTP server for emails
EMAIL_USER -> Email account username
EMAIL_PASS -> Email account password
REDIS_URL -> Redis connection string
AWS_ACCESS_KEY -> AWS access key
AWS_SECRET_KEY -> AWS secret key
```

---

## Practice Exercises

### Exercise 1

Create a new Express project

Add a .env file with

```text
PORT=5000
APP_NAME=My Express App
```

Use these variables in your server

### Exercise 2

Add validation to check if required environment variables exist

If PORT is missing, use 3000 as default

### Exercise 3

Create three different .env files

```text
.env.development
.env.staging
.env.production
```

Load the correct file based on NODE_ENV

### Exercise 4

Create a configuration object

```javascript
const config = {
  port: process.env.PORT || 3000,
  env: process.env.NODE_ENV || "development",
  jwtSecret: process.env.JWT_SECRET,
  isProduction: process.env.NODE_ENV === "production"
};
```

Use this config object throughout your app

### Exercise 5

Add a route /api/env-check

Return which environment variables are set

Do NOT return secret values

Only return non-sensitive variables

---

## Interview Questions

### What are environment variables

Environment variables are values stored outside your code that configure your application

### Why should we use environment variables

To avoid hardcoding values, keep secrets safe, and have different settings for different environments

### What is dotenv

dotenv is an npm package that loads environment variables from a .env file into process.env

### What should you never do with .env files

Never commit .env files to GitHub or share them publicly

### What is the purpose of .env.example

To show other developers what environment variables are needed without sharing actual secret values

### How do you access environment variables in Node.js

Using process.env.VARIABLE_NAME

### What happens if you change .env while the server is running

The changes do not take effect until you restart the server

### Why should you provide default values for environment variables

So your application still works if someone forgets to set a variable

---

## Summary

In this session, you learned

* What environment variables are
* Why we need environment variables
* What dotenv does
* How to install and use dotenv
* How to create a .env file
* How to load environment variables
* How to access process.env
* How to provide default values
* Different environments (development, staging, production)
* Important rules for .env files
* Creating .env.example for sharing
* Adding .env to .gitignore

Environment variables are essential for professional Node.js applications

---