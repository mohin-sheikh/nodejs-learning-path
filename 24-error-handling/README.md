## Table of Contents

* [What is Error Handling](#what-is-error-handling)
* [Types of Errors](#types-of-errors)
* [Synchronous Errors](#synchronous-errors)
* [Asynchronous Errors](#asynchronous-errors)
* [try-catch Blocks](#try-catch-blocks)
* [Custom Error Class](#custom-error-class)
* [Express Error Handling Middleware](#express-error-handling-middleware)
* [Handling Specific Errors](#handling-specific-errors)
* [Unhandled Promise Rejections](#unhandled-promise-rejections)
* [Complete Error Handling Example](#complete-error-handling-example)
* [Practice Exercises](#practice-exercises)
* [Interview Questions](#interview-questions)
* [Summary](#summary)

---

## What is Error Handling

Error handling is managing when things go wrong

Things can go wrong in many ways

```text
Database connection fails
User sends invalid data
Resource not found
Server runs out of memory
Network timeout
```

Without error handling

```text
Application crashes
User sees ugly error messages
No one knows what happened
Data might be corrupted
Bad user experience
```

With error handling

```text
Application continues running
User sees friendly error message
Error is logged for debugging
Developers get notified
Data stays safe
```

Good error handling is essential for professional applications

---

## Types of Errors

There are two main types of errors

### Operational Errors

Errors that happen during normal operation

Examples

```text
User input validation fails
Database connection lost
File not found
API rate limit exceeded
Network timeout
```

These errors are expected

We should handle them gracefully

### Programming Errors

Bugs in your code

Examples

```text
Using undefined variable
Calling non-existent function
Wrong data type
Syntax errors
Logic errors
```

These errors are unexpected

We should fix them in code

---

## Synchronous Errors

Synchronous errors happen immediately

They occur in code that runs line by line

Example of synchronous error

```javascript
app.get("/user", (req, res) => {
  const user = null;
  
  // This will cause an error
  // Cannot read property 'name' of null
  res.send(user.name);
});
```

Handling synchronous errors with try-catch

```javascript
app.get("/user", (req, res) => {
  try {
    const user = null;
    res.send(user.name);
  } catch (error) {
    res.status(500).json({
      success: false,
      message: "Something went wrong",
      error: error.message
    });
  }
});
```

---

## Asynchronous Errors

Asynchronous errors happen later

They occur in callbacks, promises, or async/await

Example with callback

```javascript
app.get("/user", (req, res) => {
  fs.readFile("user.txt", (err, data) => {
    if (err) {
      return res.status(500).json({ error: err.message });
    }
    res.json({ data });
  });
});
```

Example with promise

```javascript
app.get("/user", (req, res) => {
  User.findById(req.params.id)
    .then(user => {
      res.json(user);
    })
    .catch(error => {
      res.status(500).json({ error: error.message });
    });
});
```

Example with async/await

```javascript
app.get("/user", async (req, res) => {
  try {
    const user = await User.findById(req.params.id);
    res.json(user);
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
});
```

---

## try-catch Blocks

try-catch is used to handle errors

Basic structure

```javascript
try {
  // Code that might throw an error
  const result = someRiskyOperation();
  console.log(result);
} catch (error) {
  // Code that runs if error happens
  console.error("Error occurred:", error.message);
}
```

try-catch with different error types

```javascript
try {
  let user = null;
  
  if (!user) {
    throw new Error("User not found");
  }
  
  console.log(user.name);
  
} catch (error) {
  if (error.message === "User not found") {
    console.log("Please provide a valid user");
  } else {
    console.log("Unknown error:", error.message);
  }
}
```

try-catch-finally

```javascript
try {
  console.log("Trying to connect to database");
  await db.connect();
  console.log("Connected successfully");
} catch (error) {
  console.log("Connection failed:", error.message);
} finally {
  // This always runs, whether error or not
  console.log("Closing connection");
  await db.close();
}
```

---

## Custom Error Class

Create your own error class for better error handling

Create utils/AppError.js

```javascript
class AppError extends Error {
  constructor(message, statusCode) {
    super(message);
    this.statusCode = statusCode;
    this.status = `${statusCode}`.startsWith("4") ? "fail" : "error";
    this.isOperational = true;
    
    Error.captureStackTrace(this, this.constructor);
  }
}

module.exports = AppError;
```

Using custom error

```javascript
const AppError = require("./utils/AppError");

app.get("/user/:id", async (req, res, next) => {
  const user = await User.findById(req.params.id);
  
  if (!user) {
    // Throw custom error
    return next(new AppError("User not found", 404));
  }
  
  res.json(user);
});
```

Benefits of custom error

```text
Consistent error format
Includes status code
Easy to identify error type
Works with error handling middleware
```

---

## Express Error Handling Middleware

Express has special middleware for errors

It takes four parameters (err, req, res, next)

Create middleware/errorMiddleware.js

```javascript
const errorHandler = (err, req, res, next) => {
  // Default values
  let statusCode = err.statusCode || 500;
  let message = err.message || "Internal Server Error";
  
  // Log error for debugging
  console.error(`[ERROR] ${err.stack}`);
  
  // Send response
  res.status(statusCode).json({
    success: false,
    status: err.status || "error",
    message: message,
    // Show stack trace only in development
    ...(process.env.NODE_ENV === "development" && { stack: err.stack })
  });
};

module.exports = errorHandler;
```

Using error handler in app

```javascript
const errorHandler = require("./middleware/errorMiddleware");

// All routes go here
app.use("/api", routes);

// Error handler must be last
app.use(errorHandler);
```

---

## Handling Specific Errors

Different errors need different handling

MongoDB duplicate key error

```javascript
const handleDuplicateKeyError = (err) => {
  const field = Object.keys(err.keyPattern)[0];
  const message = `${field} already exists. Please use another value.`;
  return new AppError(message, 400);
};
```

MongoDB validation error

```javascript
const handleValidationError = (err) => {
  const errors = Object.values(err.errors).map(el => el.message);
  const message = `Invalid input: ${errors.join(". ")}`;
  return new AppError(message, 400);
};
```

JWT error

```javascript
const handleJWTError = () => {
  return new AppError("Invalid token. Please login again.", 401);
};

const handleJWTExpiredError = () => {
  return new AppError("Token expired. Please login again.", 401);
};
```

Complete error handler with specific handling

```javascript
const errorHandler = (err, req, res, next) => {
  let error = { ...err };
  error.message = err.message;
  
  // Log error
  console.error(err);
  
  // MongoDB duplicate key error
  if (err.code === 11000) {
    const field = Object.keys(err.keyPattern)[0];
    error = new AppError(`${field} already exists`, 400);
  }
  
  // MongoDB validation error
  if (err.name === "ValidationError") {
    const errors = Object.values(err.errors).map(e => e.message);
    error = new AppError(errors.join(". "), 400);
  }
  
  // JWT errors
  if (err.name === "JsonWebTokenError") {
    error = new AppError("Invalid token", 401);
  }
  
  if (err.name === "TokenExpiredError") {
    error = new AppError("Token expired", 401);
  }
  
  // Cast error (invalid MongoDB id)
  if (err.name === "CastError") {
    error = new AppError(`Invalid ${err.path}: ${err.value}`, 400);
  }
  
  res.status(error.statusCode || 500).json({
    success: false,
    message: error.message || "Server Error",
    ...(process.env.NODE_ENV === "development" && { stack: err.stack })
  });
};
```

---

## Unhandled Promise Rejections

Sometimes errors are not caught

These can crash your application

Handle them globally

```javascript
// Catch unhandled promise rejections
process.on("unhandledRejection", (err) => {
  console.log("UNHANDLED REJECTION! Shutting down...");
  console.log(err.name, err.message);
  
  // Close server gracefully
  server.close(() => {
    process.exit(1);
  });
});

// Catch uncaught exceptions
process.on("uncaughtException", (err) => {
  console.log("UNCAUGHT EXCEPTION! Shutting down...");
  console.log(err.name, err.message);
  process.exit(1);
});
```

Put these at the top of your server.js

---

## Complete Error Handling Example

Project structure

```text
error-handling-demo/
├── utils/
│   └── AppError.js
├── middleware/
│   └── errorMiddleware.js
├── models/
│   └── User.js
├── controllers/
│   └── userController.js
├── routes/
│   └── userRoutes.js
└── server.js
```

utils/AppError.js

```javascript
class AppError extends Error {
  constructor(message, statusCode) {
    super(message);
    this.statusCode = statusCode;
    this.status = `${statusCode}`.startsWith("4") ? "fail" : "error";
    this.isOperational = true;
    
    Error.captureStackTrace(this, this.constructor);
  }
}

module.exports = AppError;
```

middleware/errorMiddleware.js

```javascript
const AppError = require("../utils/AppError");

const errorHandler = (err, req, res, next) => {
  let error = { ...err };
  error.message = err.message;
  
  // Log error
  console.error("Error:", err);
  
  // Handle specific errors
  if (err.code === 11000) {
    const field = Object.keys(err.keyPattern)[0];
    error = new AppError(`${field} already exists`, 400);
  }
  
  if (err.name === "ValidationError") {
    const errors = Object.values(err.errors).map(e => e.message);
    error = new AppError(errors.join(", "), 400);
  }
  
  if (err.name === "CastError") {
    error = new AppError(`Invalid ${err.path}: ${err.value}`, 400);
  }
  
  if (err.name === "JsonWebTokenError") {
    error = new AppError("Invalid token. Please login again.", 401);
  }
  
  if (err.name === "TokenExpiredError") {
    error = new AppError("Token expired. Please login again.", 401);
  }
  
  res.status(error.statusCode || 500).json({
    success: false,
    message: error.message || "Something went wrong",
    ...(process.env.NODE_ENV === "development" && { stack: err.stack })
  });
};

// Async wrapper to avoid try-catch in controllers
const catchAsync = (fn) => {
  return (req, res, next) => {
    fn(req, res, next).catch(next);
  };
};

module.exports = { errorHandler, catchAsync };
```

controllers/userController.js

```javascript
const User = require("../models/User");
const AppError = require("../utils/AppError");
const { catchAsync } = require("../middleware/errorMiddleware");

// Get all users - with catchAsync wrapper
exports.getAllUsers = catchAsync(async (req, res, next) => {
  const users = await User.find();
  
  res.status(200).json({
    success: true,
    count: users.length,
    data: users
  });
});

// Get user by id
exports.getUserById = catchAsync(async (req, res, next) => {
  const user = await User.findById(req.params.id);
  
  if (!user) {
    return next(new AppError("User not found", 404));
  }
  
  res.status(200).json({
    success: true,
    data: user
  });
});

// Create user
exports.createUser = catchAsync(async (req, res, next) => {
  const { name, email, password } = req.body;
  
  if (!name || !email || !password) {
    return next(new AppError("Please provide name, email and password", 400));
  }
  
  const user = await User.create({ name, email, password });
  
  res.status(201).json({
    success: true,
    data: user
  });
});

// Update user
exports.updateUser = catchAsync(async (req, res, next) => {
  const user = await User.findByIdAndUpdate(req.params.id, req.body, {
    new: true,
    runValidators: true
  });
  
  if (!user) {
    return next(new AppError("User not found", 404));
  }
  
  res.status(200).json({
    success: true,
    data: user
  });
});

// Delete user
exports.deleteUser = catchAsync(async (req, res, next) => {
  const user = await User.findByIdAndDelete(req.params.id);
  
  if (!user) {
    return next(new AppError("User not found", 404));
  }
  
  res.status(200).json({
    success: true,
    message: "User deleted successfully"
  });
});

// Route that intentionally throws error
exports.throwError = catchAsync(async (req, res, next) => {
  // This will be caught by catchAsync
  throw new AppError("This is a test error", 400);
});
```

server.js

```javascript
require("dotenv").config();
const express = require("express");
const mongoose = require("mongoose");

const userRoutes = require("./routes/userRoutes");
const { errorHandler } = require("./middleware/errorMiddleware");

const app = express();
app.use(express.json());

// Routes
app.use("/api/users", userRoutes);

// 404 for undefined routes
app.all("*", (req, res, next) => {
  next(new AppError(`Route ${req.originalUrl} not found`, 404));
});

// Error handling middleware (MUST be last)
app.use(errorHandler);

// Connect to database and start server
const PORT = process.env.PORT || 5000;

mongoose.connect(process.env.MONGODB_URI)
  .then(() => {
    console.log("Connected to MongoDB");
    
    const server = app.listen(PORT, () => {
      console.log(`Server running on port ${PORT}`);
    });
    
    // Handle unhandled promise rejections
    process.on("unhandledRejection", (err) => {
      console.log("UNHANDLED REJECTION! Shutting down...");
      console.log(err.name, err.message);
      server.close(() => {
        process.exit(1);
      });
    });
    
    // Handle uncaught exceptions
    process.on("uncaughtException", (err) => {
      console.log("UNCAUGHT EXCEPTION! Shutting down...");
      console.log(err.name, err.message);
      process.exit(1);
    });
    
  })
  .catch(err => {
    console.error("MongoDB connection failed", err);
    process.exit(1);
  });
```

---

## Testing Error Handling

Test different error scenarios

```bash
# Route not found
curl http://localhost:5000/api/invalid

# User not found
curl http://localhost:5000/api/users/999999999999999999999999

# Invalid user id format
curl http://localhost:5000/api/users/invalid-id

# Missing required fields
curl -X POST http://localhost:5000/api/users \
  -H "Content-Type: application/json" \
  -d '{"name": "John"}'

# Duplicate email
curl -X POST http://localhost:5000/api/users \
  -H "Content-Type: application/json" \
  -d '{"name":"John","email":"existing@test.com","password":"123456"}'

# Test error route
curl http://localhost:5000/api/users/test-error
```

---

## Practice Exercises

### Exercise 1

Create a custom error for "ResourceNotFound"

Use status code 404

### Exercise 2

Add error handling for file upload

Handle cases where file is too large or wrong type

### Exercise 3

Create a rate limit exceeded error

Return status 429 when user makes too many requests

### Exercise 4

Add validation error handler for Zod or Joi validation

### Exercise 5

Create a logger that writes errors to a file

Also sends email for critical errors

---

## Interview Questions

### What is error handling

Error handling is managing when things go wrong in an application

### What is the difference between operational and programming errors

Operational errors are expected (validation, network)
Programming errors are bugs in code

### How do you handle synchronous errors

Using try-catch blocks

### How do you handle asynchronous errors

Using .catch() with promises or try-catch with async/await

### What is Express error handling middleware

Special middleware with 4 parameters (err, req, res, next) that handles errors

### Why should error handler be last middleware

So it can catch errors from all routes and middleware before it

### What is unhandledRejection

Promise rejection that was not caught with .catch()

### What is uncaughtException

Error thrown in code that was not caught with try-catch

### What is the purpose of catchAsync wrapper

To avoid writing try-catch in every controller

---

## Summary

In this session, you learned

* What error handling is
* Types of errors (operational vs programming)
* How to handle synchronous errors with try-catch
* How to handle asynchronous errors
* Creating custom error classes
* Express error handling middleware
* Handling specific errors (MongoDB, JWT)
* Unhandled rejections and exceptions
* Complete error handling system

Good error handling makes your application reliable and professional

---