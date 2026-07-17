## Table of Contents

* [Why Project Structure Matters](#why-project-structure-matters)
* [Bad Project Structure](#bad-project-structure)
* [Good Project Structure](#good-project-structure)
* [Folder by Feature vs Folder by Type](#folder-by-feature-vs-folder-by-type)
* [Complete Project Structure](#complete-project-structure)
* [Environment Configuration](#environment-configuration)
* [Centralized Error Handling](#centralized-error-handling)
* [Response Helpers](#response-helpers)
* [Constants and Enums](#constants-and-enums)
* [Utilities and Helpers](#utilities-and-helpers)
* [Naming Conventions](#naming-conventions)
* [Practice Exercises](#practice-exercises)
* [Interview Questions](#interview-questions)
* [Summary](#summary)

---

## What is Project Structure

Project structure is how you organize your code files and folders

Think of it like organizing a house

Bad organization

```text
Everything in one room
Clothes in kitchen
Books in bathroom
Food in bedroom
Hard to find anything
```

Good organization

```text
Kitchen for food
Bedroom for clothes
Living room for entertainment
Bathroom for toiletries
Easy to find everything
```

Same with code

Good structure makes your project easy to understand

---

## Why Project Structure Matters

Without good structure

```text
Files scattered everywhere
Hard to find what you need
Duplicate code everywhere
New developers get confused
Bugs are hard to track
Scaling is impossible
```

With good structure

```text
Every file has its place
Easy to find anything
Code is reusable
New developers learn quickly
Bugs are easy to fix
Application scales easily
```

Benefits of good structure

```text
Maintainability -> Easy to update and fix
Scalability -> Easy to add new features
Team collaboration -> Everyone knows where things go
Code reusability -> No duplicate code
Onboarding -> New developers understand quickly
Testing -> Easy to write and run tests
```

---

## Bad Project Structure

Here is what NOT to do

```text
project/
│
├── server.js (3000 lines)
├── database.js
├── auth.js
├── utils.js
├── models.js
├── routes.js
├── controllers.js
├── middleware.js
└── config.js
```

Problems with this structure

```text
Files are too large
No separation of concerns
Everything is mixed together
Hard to find specific code
Cannot reuse code easily
Difficult to test
```

Another bad example

```text
project/
│
├── models/
│   ├── userModel.js
│   ├── productModel.js
│   ├── orderModel.js
│   └── categoryModel.js
│
├── controllers/
│   ├── userController.js
│   ├── productController.js
│   ├── orderController.js
│   └── categoryController.js
│
├── routes/
│   ├── userRoutes.js
│   ├── productRoutes.js
│   ├── orderRoutes.js
│   └── categoryRoutes.js
```

This is better but still has problems

```text
When adding a feature, you touch 3 folders
Related code is separated
Hard to delete a feature
Not modular
```

---

## Good Project Structure

Here is a professional project structure

```text
project/
│
├── src/
│   ├── config/
│   │   ├── database.js
│   │   ├── redis.js
│   │   └── cloudinary.js
│   │
│   ├── controllers/
│   │   ├── authController.js
│   │   ├── userController.js
│   │   └── productController.js
│   │
│   ├── middleware/
│   │   ├── authMiddleware.js
│   │   ├── errorMiddleware.js
│   │   ├── validationMiddleware.js
│   │   └── rateLimitMiddleware.js
│   │
│   ├── models/
│   │   ├── User.js
│   │   ├── Product.js
│   │   └── Order.js
│   │
│   ├── routes/
│   │   ├── authRoutes.js
│   │   ├── userRoutes.js
│   │   ├── productRoutes.js
│   │   └── index.js
│   │
│   ├── services/
│   │   ├── authService.js
│   │   ├── emailService.js
│   │   └── fileUploadService.js
│   │
│   ├── utils/
│   │   ├── AppError.js
│   │   ├── catchAsync.js
│   │   ├── responseHelper.js
│   │   └── validators.js
│   │
│   ├── constants/
│   │   ├── statusCodes.js
│   │   ├── errorMessages.js
│   │   └── roles.js
│   │
│   ├── validation/
│   │   ├── userValidation.js
│   │   └── productValidation.js
│   │
│   └── app.js
│
├── tests/
│   ├── unit/
│   ├── integration/
│   └── fixtures/
│
├── logs/
├── uploads/
│
├── .env
├── .env.example
├── .gitignore
├── package.json
└── server.js
```

---

## Folder by Feature vs Folder by Type

Two main approaches to organizing code

Folder by Type (what we showed above)

```text
All controllers together
All models together
All routes together
```

Pros

```text
Easy to find all controllers
Good for small to medium projects
Familiar to most developers
```

Cons

```text
When adding a feature, you touch multiple folders
Code for one feature is scattered
Hard to delete a feature
```

Folder by Feature (modular approach)

```text
src/
│
├── features/
│   ├── auth/
│   │   ├── authController.js
│   │   ├── authRoutes.js
│   │   ├── authService.js
│   │   ├── authValidation.js
│   │   └── auth.test.js
│   │
│   ├── users/
│   │   ├── userController.js
│   │   ├── userRoutes.js
│   │   ├── userModel.js
│   │   ├── userService.js
│   │   └── user.test.js
│   │
│   └── products/
│       ├── productController.js
│       ├── productRoutes.js
│       ├── productModel.js
│       ├── productService.js
│       └── product.test.js
```

Pros

```text
All code for a feature is together
Easy to delete a feature
Better for large projects
More modular
```

Cons

```text
More complex setup
Shared code needs careful management
Overkill for small projects
```

For beginners, use Folder by Type

For large projects, use Folder by Feature

---

## Complete Project Structure

Let me show you the complete professional structure

```text
my-express-app/
│
├── src/
│   ├── config/
│   │   └── database.js
│   │
│   ├── controllers/
│   │   ├── authController.js
│   │   └── userController.js
│   │
│   ├── middleware/
│   │   ├── authMiddleware.js
│   │   ├── errorMiddleware.js
│   │   └── validationMiddleware.js
│   │
│   ├── models/
│   │   ├── User.js
│   │   └── Token.js
│   │
│   ├── routes/
│   │   ├── authRoutes.js
│   │   ├── userRoutes.js
│   │   └── index.js
│   │
│   ├── services/
│   │   ├── authService.js
│   │   └── emailService.js
│   │
│   ├── utils/
│   │   ├── AppError.js
│   │   ├── catchAsync.js
│   │   └── sendEmail.js
│   │
│   ├── constants/
│   │   ├── statusCodes.js
│   │   └── roles.js
│   │
│   └── app.js
│
├── tests/
│   ├── unit/
│   └── integration/
│
├── .env
├── .env.example
├── .gitignore
├── package.json
└── server.js
```

---

## Environment Configuration

config/database.js

```javascript
const mongoose = require("mongoose");

const connectDB = async () => {
  try {
    const conn = await mongoose.connect(process.env.MONGODB_URI);
    console.log(`MongoDB Connected: ${conn.connection.host}`);
  } catch (error) {
    console.error(`Error: ${error.message}`);
    process.exit(1);
  }
};

module.exports = connectDB;
```

config/redis.js (if using Redis)

```javascript
const redis = require("redis");

const redisClient = redis.createClient({
  url: process.env.REDIS_URL
});

redisClient.on("connect", () => {
  console.log("Redis connected");
});

redisClient.on("error", (err) => {
  console.log("Redis error:", err);
});

module.exports = redisClient;
```

.env file

```text
# Server
NODE_ENV=development
PORT=5000

# Database
MONGODB_URI=mongodb+srv://user:pass@cluster.mongodb.net/
DB_NAME=myapp

# JWT
JWT_SECRET=my-super-secret-key
JWT_EXPIRE=7d

# Email
EMAIL_HOST=smtp.gmail.com
EMAIL_PORT=587
EMAIL_USER=admin@example.com
EMAIL_PASS=password

# Redis
REDIS_URL=redis://localhost:6379
```

---

## Centralized Error Handling

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

utils/catchAsync.js

```javascript
const catchAsync = (fn) => {
  return (req, res, next) => {
    fn(req, res, next).catch(next);
  };
};

module.exports = catchAsync;
```

middleware/errorMiddleware.js

```javascript
const AppError = require("../utils/AppError");

const handleCastErrorDB = (err) => {
  const message = `Invalid ${err.path}: ${err.value}`;
  return new AppError(message, 400);
};

const handleDuplicateFieldsDB = (err) => {
  const value = err.errmsg.match(/(["'])(\\?.)*?\1/)[0];
  const message = `Duplicate field value: ${value}. Please use another value`;
  return new AppError(message, 400);
};

const handleValidationErrorDB = (err) => {
  const errors = Object.values(err.errors).map(el => el.message);
  const message = `Invalid input data. ${errors.join(". ")}`;
  return new AppError(message, 400);
};

const sendErrorDev = (err, res) => {
  res.status(err.statusCode).json({
    status: err.status,
    error: err,
    message: err.message,
    stack: err.stack
  });
};

const sendErrorProd = (err, res) => {
  if (err.isOperational) {
    res.status(err.statusCode).json({
      status: err.status,
      message: err.message
    });
  } else {
    console.error("ERROR 💥", err);
    
    res.status(500).json({
      status: "error",
      message: "Something went wrong"
    });
  }
};

module.exports = (err, req, res, next) => {
  err.statusCode = err.statusCode || 500;
  err.status = err.status || "error";
  
  if (process.env.NODE_ENV === "development") {
    sendErrorDev(err, res);
  } else {
    let error = { ...err };
    error.message = err.message;
    
    if (error.name === "CastError") error = handleCastErrorDB(error);
    if (error.code === 11000) error = handleDuplicateFieldsDB(error);
    if (error.name === "ValidationError") error = handleValidationErrorDB(error);
    
    sendErrorProd(error, res);
  }
};
```

---

## Response Helpers

utils/responseHelper.js

```javascript
class ResponseHelper {
  static success(res, data, message = "Success", statusCode = 200) {
    return res.status(statusCode).json({
      success: true,
      message,
      data
    });
  }
  
  static error(res, message, statusCode = 400) {
    return res.status(statusCode).json({
      success: false,
      message
    });
  }
  
  static created(res, data, message = "Resource created successfully") {
    return this.success(res, data, message, 201);
  }
  
  static notFound(res, message = "Resource not found") {
    return this.error(res, message, 404);
  }
  
  static badRequest(res, message = "Bad request") {
    return this.error(res, message, 400);
  }
  
  static unauthorized(res, message = "Unauthorized") {
    return this.error(res, message, 401);
  }
  
  static forbidden(res, message = "Forbidden") {
    return this.error(res, message, 403);
  }
}

module.exports = ResponseHelper;
```

Using response helper in controllers

```javascript
const ResponseHelper = require("../utils/responseHelper");

exports.getAllUsers = catchAsync(async (req, res, next) => {
  const users = await User.find();
  
  ResponseHelper.success(res, users, "Users fetched successfully");
});

exports.createUser = catchAsync(async (req, res, next) => {
  const user = await User.create(req.body);
  
  ResponseHelper.created(res, user, "User created successfully");
});

exports.getUserById = catchAsync(async (req, res, next) => {
  const user = await User.findById(req.params.id);
  
  if (!user) {
    return ResponseHelper.notFound(res, "User not found");
  }
  
  ResponseHelper.success(res, user);
});
```

---

## Constants and Enums

constants/statusCodes.js

```javascript
const HTTP_STATUS = {
  OK: 200,
  CREATED: 201,
  ACCEPTED: 202,
  NO_CONTENT: 204,
  BAD_REQUEST: 400,
  UNAUTHORIZED: 401,
  FORBIDDEN: 403,
  NOT_FOUND: 404,
  CONFLICT: 409,
  UNPROCESSABLE_ENTITY: 422,
  TOO_MANY_REQUESTS: 429,
  INTERNAL_SERVER_ERROR: 500,
  BAD_GATEWAY: 502,
  SERVICE_UNAVAILABLE: 503
};

module.exports = HTTP_STATUS;
```

constants/roles.js

```javascript
const USER_ROLES = {
  ADMIN: "admin",
  USER: "user",
  MODERATOR: "moderator",
  GUEST: "guest"
};

const ROLE_PERMISSIONS = {
  [USER_ROLES.ADMIN]: ["create", "read", "update", "delete"],
  [USER_ROLES.MODERATOR]: ["create", "read", "update"],
  [USER_ROLES.USER]: ["read", "update_own"],
  [USER_ROLES.GUEST]: ["read"]
};

module.exports = { USER_ROLES, ROLE_PERMISSIONS };
```

constants/errorMessages.js

```javascript
const ERROR_MESSAGES = {
  // Auth errors
  INVALID_CREDENTIALS: "Invalid email or password",
  UNAUTHORIZED: "You are not logged in",
  TOKEN_EXPIRED: "Token expired. Please login again",
  INVALID_TOKEN: "Invalid token",
  
  // User errors
  USER_NOT_FOUND: "User not found",
  EMAIL_EXISTS: "Email already exists",
  
  // Validation errors
  MISSING_FIELDS: "Missing required fields",
  INVALID_EMAIL: "Please provide a valid email",
  WEAK_PASSWORD: "Password must be at least 6 characters",
  
  // Database errors
  DUPLICATE_ENTRY: "Duplicate entry found",
  DATABASE_ERROR: "Database error occurred",
  
  // Server errors
  INTERNAL_ERROR: "Internal server error",
  SERVICE_UNAVAILABLE: "Service temporarily unavailable"
};

module.exports = ERROR_MESSAGES;
```

---

## Utilities and Helpers

utils/validators.js

```javascript
const validateEmail = (email) => {
  const regex = /^\w+([\.-]?\w+)*@\w+([\.-]?\w+)*(\.\w{2,3})+$/;
  return regex.test(email);
};

const validatePhoneNumber = (phone) => {
  const regex = /^\d{10}$/;
  return regex.test(phone);
};

const validatePassword = (password) => {
  return password.length >= 6;
};

const validateObjectId = (id) => {
  const mongoose = require("mongoose");
  return mongoose.Types.ObjectId.isValid(id);
};

module.exports = {
  validateEmail,
  validatePhoneNumber,
  validatePassword,
  validateObjectId
};
```

utils/dateHelpers.js

```javascript
const formatDate = (date, format = "YYYY-MM-DD") => {
  const d = new Date(date);
  const year = d.getFullYear();
  const month = String(d.getMonth() + 1).padStart(2, "0");
  const day = String(d.getDate()).padStart(2, "0");
  
  if (format === "YYYY-MM-DD") {
    return `${year}-${month}-${day}`;
  }
  
  return d.toLocaleDateString();
};

const getAge = (birthDate) => {
  const today = new Date();
  const birth = new Date(birthDate);
  let age = today.getFullYear() - birth.getFullYear();
  const monthDiff = today.getMonth() - birth.getMonth();
  
  if (monthDiff < 0 || (monthDiff === 0 && today.getDate() < birth.getDate())) {
    age--;
  }
  
  return age;
};

module.exports = { formatDate, getAge };
```

---

## Main App File

src/app.js

```javascript
const express = require("express");
const cors = require("cors");
const helmet = require("helmet");
const morgan = require("morgan");
const rateLimit = require("express-rate-limit");

const authRoutes = require("./routes/authRoutes");
const userRoutes = require("./routes/userRoutes");
const errorMiddleware = require("./middleware/errorMiddleware");

const app = express();

// Security middleware
app.use(helmet());
app.use(cors());

// Rate limiting
const limiter = rateLimit({
  windowMs: 15 * 60 * 1000, // 15 minutes
  max: 100, // limit each IP to 100 requests per windowMs
  message: "Too many requests from this IP"
});
app.use("/api", limiter);

// Body parsing middleware
app.use(express.json({ limit: "10mb" }));
app.use(express.urlencoded({ extended: true, limit: "10mb" }));

// Logging
if (process.env.NODE_ENV === "development") {
  app.use(morgan("dev"));
}

// Static files
app.use("/uploads", express.static("uploads"));

// Routes
app.use("/api/auth", authRoutes);
app.use("/api/users", userRoutes);

// Health check
app.get("/health", (req, res) => {
  res.status(200).json({ status: "OK", timestamp: new Date() });
});

// 404 handler
app.all("*", (req, res, next) => {
  next(new AppError(`Route ${req.originalUrl} not found`, 404));
});

// Global error handler
app.use(errorMiddleware);

module.exports = app;
```

server.js

```javascript
const dotenv = require("dotenv");
dotenv.config();

const app = require("./src/app");
const connectDB = require("./src/config/database");

// Handle uncaught exceptions
process.on("uncaughtException", (err) => {
  console.log("UNCAUGHT EXCEPTION! Shutting down...");
  console.log(err.name, err.message);
  process.exit(1);
});

// Connect to database
connectDB();

// Start server
const PORT = process.env.PORT || 5000;
const server = app.listen(PORT, () => {
  console.log(`Server running in ${process.env.NODE_ENV} mode on port ${PORT}`);
});

// Handle unhandled promise rejections
process.on("unhandledRejection", (err) => {
  console.log("UNHANDLED REJECTION! Shutting down...");
  console.log(err.name, err.message);
  server.close(() => {
    process.exit(1);
  });
});
```

---

## Naming Conventions

File naming

```text
User.js -> Models (PascalCase)
userController.js -> Controllers (camelCase)
userRoutes.js -> Routes (camelCase)
authMiddleware.js -> Middleware (camelCase)
AppError.js -> Utils (PascalCase)
```

Variable naming

```text
const userName = "John" -> camelCase for variables
const MAX_SIZE = 100 -> UPPER_SNAKE_CASE for constants
class UserController -> PascalCase for classes
```

Function naming

```text
getUserById() -> Verb + noun
createUser() -> Verb + noun
handleError() -> Handle + noun
```

Route naming

```text
/api/users -> plural nouns
/api/users/:id -> with parameter
/api/users/search -> action at end
```

Database naming

```text
users collection -> plural lowercase
user_id field -> snake_case
created_at field -> snake_case
```

---

## Practice Exercises

### Exercise 1

Take a messy project and refactor it

Use the structure shown above

### Exercise 2

Add response helper to your project

Use it in all controllers

### Exercise 3

Create constants for status codes

Use them everywhere instead of writing 200, 404

### Exercise 4

Implement centralized error handling

Use the errorMiddleware pattern

### Exercise 5

Create a validation helper

Reuse it across multiple controllers

---

## Interview Questions

### Why is project structure important

It makes code maintainable, scalable, and easy to understand

### What is the difference between folder by type and folder by feature

Folder by type groups by file type (controllers, models)
Folder by feature groups by business feature (auth, users)

### What should go in utils folder

Helper functions that are used across the application

### What is the purpose of constants folder

To store reusable values like status codes and error messages

### Why use centralized error handling

To handle all errors in one place and avoid duplicate code

### What is the benefit of response helper

To standardize API responses across the application

### Where should database connection code go

In config folder or database.js file

### What should be in app.js vs server.js

app.js contains Express configuration
server.js starts the server and connects to database

---

## Summary

In this session, you learned

* Why project structure matters
* Bad vs good project structure
* Folder by type vs folder by feature
* Complete professional project structure
* Environment configuration
* Centralized error handling
* Response helpers
* Constants and enums
* Utilities and helpers
* Naming conventions

Good project structure makes your application professional and maintainable

---