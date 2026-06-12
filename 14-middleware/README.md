## Table of Contents

* [What is Middleware](#what-is-middleware)
* [How Middleware Works](#how-middleware-works)
* [Creating Your First Middleware](#creating-your-first-middleware)
* [Application Level Middleware](#application-level-middleware)
* [Route Level Middleware](#route-level-middleware)
* [Built-in Middleware](#built-in-middleware)
* [Third Party Middleware](#third-party-middleware)
* [Logger Middleware Example](#logger-middleware-example)
* [Authentication Middleware Example](#authentication-middleware-example)
* [Multiple Middleware Functions](#multiple-middleware-functions)
* [Global vs Specific Middleware](#global-vs-specific-middleware)
* [Practice Exercises](#practice-exercises)
* [Interview Questions](#interview-questions)
* [Summary](#summary)

---

## What is Middleware

Middleware is a function that runs between the request and response

Think of it like a security checkpoint

```text
Request comes in
    |
    v
Middleware function runs
    |
    v
Next middleware runs
    |
    v
Route handler runs
    |
    v
Response goes out
```

Every middleware function has access to

```text
req -> Request object
res -> Response object
next -> Next function
```

The next function tells Express to move to the next middleware

Without next(), the request will stop

---

## How Middleware Works

Here is a simple diagram

```text
Client Request
      |
      v
[ Middleware 1 ] -> Does something -> next()
      |
      v
[ Middleware 2 ] -> Does something -> next()
      |
      v
[ Route Handler ] -> Sends response
      |
      v
Client Response
```

Middleware can

```text
Log requests
Check authentication
Parse data
Handle errors
Add data to req object
End the request early
```

---

## Creating Your First Middleware

A middleware function looks like this

```javascript
function myMiddleware(req, res, next) {
  console.log("Middleware running");
  next();
}
```

To use it, add it to your app

```javascript
const express = require("express");
const app = express();

// This middleware runs for every request
app.use(myMiddleware);

app.get("/", (req, res) => {
  res.send("Home Page");
});

app.listen(3000);
```

Every time someone visits any route

```text
Middleware runs first
Then the route handler runs
```

You will see "Middleware running" in the terminal for every request

---

## Application Level Middleware

Application level middleware runs for every request

Use app.use()

```javascript
const express = require("express");
const app = express();

// This runs for every request
app.use((req, res, next) => {
  console.log(`Request made to ${req.url}`);
  next();
});

app.get("/", (req, res) => {
  res.send("Home");
});

app.get("/about", (req, res) => {
  res.send("About");
});

app.listen(3000);
```

When someone visits /about

```text
Console shows "Request made to /about"
Then sends "About" response
```

This is useful for logging

---

## Route Level Middleware

Route level middleware runs only for specific routes

```javascript
const express = require("express");
const app = express();

const logMiddleware = (req, res, next) => {
  console.log(`Logging ${req.method} request to ${req.url}`);
  next();
};

// This middleware only runs for /students routes
app.use("/students", logMiddleware);

app.get("/students", (req, res) => {
  res.json({ message: "Students list" });
});

app.get("/teachers", (req, res) => {
  // logMiddleware does NOT run for this route
  res.json({ message: "Teachers list" });
});

app.listen(3000);
```

Now logMiddleware only runs when someone visits any route starting with /students

---

## Built-in Middleware

Express comes with some built-in middleware

### express.json()

Parses JSON data from request body

```javascript
app.use(express.json());

app.post("/user", (req, res) => {
  console.log(req.body.name);
  res.send("User received");
});
```

Without this, req.body would be undefined

### express.urlencoded()

Parses form data from HTML forms

```javascript
app.use(express.urlencoded({ extended: true }));

app.post("/form", (req, res) => {
  console.log(req.body);
  res.send("Form data received");
});
```

### express.static()

Serves static files like HTML, CSS, images

```javascript
app.use(express.static("public"));

// Now any file in public folder can be accessed
// http://localhost:3000/style.css
// http://localhost:3000/logo.png
```

Create a public folder with your files

---

## Third Party Middleware

You can install middleware from npm

Popular third party middleware

```text
morgan -> Logging
cors -> Cross origin requests
helmet -> Security
rate-limit -> Rate limiting
```

Install morgan for logging

```bash
npm install morgan
```

Use it in your app

```javascript
const express = require("express");
const morgan = require("morgan");
const app = express();

app.use(morgan("tiny"));

app.get("/", (req, res) => {
  res.send("Home Page");
});

app.listen(3000);
```

Now every request is automatically logged

Output example

```text
GET / 200 - 2ms
GET /about 200 - 1ms
POST /users 201 - 5ms
```

---

## Logger Middleware Example

Create your own logger middleware

```javascript
const express = require("express");
const app = express();

const logger = (req, res, next) => {
  const timestamp = new Date().toISOString();
  console.log(`[${timestamp}] ${req.method} ${req.url}`);
  next();
};

app.use(logger);

app.get("/", (req, res) => {
  res.send("Home Page");
});

app.get("/students", (req, res) => {
  res.json({ message: "Students" });
});

app.listen(3000);
```

Terminal output when someone visits /students

```text
[2024-01-15T10:30:00.000Z] GET /students
```

This helps you track what users are doing

---

## Authentication Middleware Example

Create middleware to check if user is logged in

```javascript
const express = require("express");
const app = express();

// Fake authentication middleware
const isAuthenticated = (req, res, next) => {
  const token = req.headers.authorization;
  
  if (token === "my-secret-token") {
    next(); // User is authenticated, continue
  } else {
    res.status(401).json({ message: "Not authorized" });
  }
};

// Public route - no authentication needed
app.get("/", (req, res) => {
  res.send("Welcome to our website");
});

// Protected route - needs authentication
app.get("/dashboard", isAuthenticated, (req, res) => {
  res.json({ message: "Welcome to your dashboard" });
});

// Admin route - needs authentication
app.get("/admin", isAuthenticated, (req, res) => {
  res.json({ message: "Admin panel" });
});

app.listen(3000);
```

Now to access /dashboard or /admin

You must send a request with header

```text
Authorization: my-secret-token
```

Otherwise you get "Not authorized"

---

## Multiple Middleware Functions

You can add multiple middleware for one route

They run in order

```javascript
const express = require("express");
const app = express();

const middleware1 = (req, res, next) => {
  console.log("Middleware 1 running");
  next();
};

const middleware2 = (req, res, next) => {
  console.log("Middleware 2 running");
  next();
};

const middleware3 = (req, res, next) => {
  console.log("Middleware 3 running");
  next();
};

// All three middleware run before the route handler
app.get("/protected", middleware1, middleware2, middleware3, (req, res) => {
  res.send("All middleware ran");
});

app.listen(3000);
```

When someone visits /protected

```text
Console shows
Middleware 1 running
Middleware 2 running
Middleware 3 running
Then sends "All middleware ran"
```

---

## Global vs Specific Middleware

Global middleware runs for every route

```javascript
app.use((req, res, next) => {
  console.log("This runs for every route");
  next();
});
```

Specific middleware runs only for certain routes

```javascript
// Only runs for /students routes
app.use("/students", (req, res, next) => {
  console.log("Only for students");
  next();
});

// Only runs for this specific route
app.get("/admin", (req, res, next) => {
  console.log("Only for admin");
  next();
}, (req, res) => {
  res.send("Admin page");
});
```

---

## Complete Example with Multiple Middleware

```javascript
const express = require("express");
const app = express();

// Global middleware - runs for every request
app.use((req, res, next) => {
  console.log(`[1] Global middleware`);
  next();
});

// Another global middleware
app.use((req, res, next) => {
  console.log(`[2] Logger: ${req.method} ${req.url}`);
  next();
});

// Built-in middleware
app.use(express.json());

// Route specific middleware
const checkApiKey = (req, res, next) => {
  const apiKey = req.headers["x-api-key"];
  
  if (apiKey === "12345") {
    console.log(`[3] API key valid`);
    next();
  } else {
    res.status(401).json({ message: "Invalid API key" });
  }
};

// Public route - no API key needed
app.get("/", (req, res) => {
  res.send("Home Page");
});

// Protected route - needs API key
app.get("/data", checkApiKey, (req, res) => {
  res.json({ message: "Secret data" });
});

// Route with multiple middleware
const logTime = (req, res, next) => {
  req.requestTime = new Date().toISOString();
  next();
};

const logUser = (req, res, next) => {
  console.log(`User requested at ${req.requestTime}`);
  next();
};

app.get("/profile", logTime, logUser, (req, res) => {
  res.json({ message: "Profile page", time: req.requestTime });
});

app.listen(3000, () => {
  console.log("Server running on port 3000");
});
```

---

## Middleware Flow Summary

```text
Request comes in
      |
      v
Global Middleware 1 runs
      |
      v
Global Middleware 2 runs
      |
      v
Route specific middleware (if any)
      |
      v
Route handler runs
      |
      v
Response sent back
```

---

## Practice Exercises

### Exercise 1

Create a middleware that logs

```text
Request method
Request URL
Timestamp
```

Use this middleware for all routes

### Exercise 2

Create an authentication middleware

It should check for a header called "auth-token"

If token is "secret123", allow access
If not, return 401 with message "Invalid token"

### Exercise 3

Create a middleware that adds a property to req object

```text
req.user = { name: "John", role: "admin" }
```

Then access this in the route handler

### Exercise 4

Create a rate limiting middleware

It should allow only 3 requests per minute from the same IP

After 3 requests, return 429 with message "Too many requests"

### Exercise 5

Use morgan middleware for logging

Install morgan and add it to your app

Compare your custom logger with morgan

---

## Interview Questions

### What is middleware in Express

Middleware is a function that runs between the request and response

### What parameters does a middleware function receive

req, res, and next

### What does the next() function do

It tells Express to move to the next middleware or route handler

### What happens if you don't call next() in middleware

The request will hang and never complete

### What is the difference between app.use() and app.get()

app.use() runs middleware for all HTTP methods
app.get() runs only for GET requests

### What are some built-in middleware in Express

express.json()
express.urlencoded()
express.static()

### What is third party middleware

Middleware created by other developers that you install via npm

Examples: morgan, cors, helmet

---

## Summary

In this session, you learned

* What middleware is
* How middleware works with next()
* Application level middleware
* Route level middleware
* Built-in middleware like express.json()
* Third party middleware like morgan
* How to create custom middleware
* How to use multiple middleware functions
* The difference between global and specific middleware

Middleware is one of the most important concepts in Express

---