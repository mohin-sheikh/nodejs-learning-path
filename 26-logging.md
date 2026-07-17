## Table of Contents

* [What is Logging](#what-is-logging)
* [Why Logging is Important](#why-logging-is-important)
* [Types of Logs](#types-of-logs)
* [What is Morgan](#what-is-morgan)
* [Installing Morgan](#installing-morgan)
* [Morgan Log Formats](#morgan-log-formats)
* [Using Morgan in Express](#using-morgan-in-express)
* [Writing Logs to File](#writing-logs-to-file)
* [What is Winston](#what-is-winston)
* [Installing Winston](#installing-winston)
* [Creating a Logger with Winston](#creating-a-logger-with-winston)
* [Log Levels](#log-levels)
* [Transports in Winston](#transports-in-winston)
* [Complete Logging System](#complete-logging-system)
* [Practice Exercises](#practice-exercises)
* [Interview Questions](#interview-questions)
* [Summary](#summary)

---

## What is Logging

Logging is recording what happens in your application

Think of it like a diary

```text
Application does something
Write it down in a log file
Later you can read what happened
```

Examples of logs

```text
User John logged in at 10:30 AM
Database query took 0.5 seconds
Error: Could not connect to database
User requested page /students
File uploaded successfully
```

Logs help you understand

```text
What your application is doing
When errors occur
Who is using your application
How long things take
What needs improvement
```

---

## Why Logging is Important

Without logging

```text
Application crashes
You have no idea why
User complains
You cannot find the problem
Same error happens again
```

With logging

```text
Application crashes
You check the logs
See exactly what went wrong
Fix the problem
Prevent it from happening again
```

Benefits of logging

```text
Debugging -> Find and fix errors
Monitoring -> See if app is healthy
Security -> Detect suspicious activity
Performance -> Find slow operations
Audit -> Track who did what
Compliance -> Meet legal requirements
```

---

## Types of Logs

Different types of logs for different purposes

Request Logs

```text
What URLs were visited
What HTTP methods were used
What status codes were returned
How long requests took
```

Example

```text
GET /students 200 15ms
POST /students 201 45ms
GET /students/999 404 5ms
```

Error Logs

```text
When errors happen
What the error was
Where the error occurred
Stack trace
```

Example

```text
[ERROR] Database connection failed
[ERROR] User not found with id 999
[ERROR] Invalid email format
```

Application Logs

```text
When app starts or stops
When features are used
When important events happen
```

Example

```text
[INFO] Server started on port 3000
[INFO] Connected to MongoDB
[INFO] User john@example.com registered
```

Security Logs

```text
Failed login attempts
Unauthorized access attempts
Sensitive operations
```

Example

```text
[WARN] Failed login attempt for user admin
[WARN] Unauthorized access to /admin
[INFO] Password changed for user john
```

Performance Logs

```text
Slow database queries
Slow API responses
Resource usage
```

Example

```text
[PERF] Query took 2.5 seconds
[PERF] /api/report took 5.1 seconds
[PERF] Memory usage 85%
```

---

## What is Morgan

Morgan is HTTP request logger middleware for Express

It automatically logs every request

Why use Morgan

```text
No need to write log code manually
Many log formats available
Can log to console or file
Shows request duration
Very easy to use
```

Morgan log format example

```text
GET /students 200 15ms - 1.2KB
POST /students 201 45ms - 0.5KB
GET /students/999 404 5ms - 0.2KB
```

---

## Installing Morgan

Create a new project

```bash
mkdir logging-demo
cd logging-demo
npm init -y
```

Install packages

```bash
npm install express morgan
```

Create server.js

```javascript
const express = require("express");
const morgan = require("morgan");

const app = express();

// Use morgan for logging
app.use(morgan("tiny"));

app.get("/", (req, res) => {
  res.send("Home Page");
});

app.get("/students", (req, res) => {
  res.json({ students: [] });
});

app.listen(3000, () => {
  console.log("Server running on port 3000");
});
```

Make some requests and see the logs in terminal

---

## Morgan Log Formats

Morgan comes with several predefined formats

combined format (most detailed)

```text
:remote-addr - :remote-user [:date[clf]] ":method :url HTTP/:http-version" :status :res[content-length] ":referrer" ":user-agent"
```

Example output

```text
::1 - - [15/Jan/2024:10:30:00 +0000] "GET /students HTTP/1.1" 200 1234 "-" "Mozilla/5.0"
```

common format

```text
:remote-addr - :remote-user [:date[clf]] ":method :url HTTP/:http-version" :status :res[content-length]
```

dev format (color coded for development)

```text
:method :url :status :response-time ms - :res[content-length]
```

Example output (with colors)

```text
GET /students 200 15ms - 1.2KB
POST /students 201 45ms - 0.5KB
GET /students/999 404 5ms - 0.2KB
```

short format

```text
:remote-addr :method :url HTTP/:http-version :status :res[content-length] - :response-time ms
```

tiny format (smallest)

```text
:method :url :status :res[content-length] - :response-time ms
```

Example output

```text
GET /students 200 1234 - 15ms
```

Using different formats

```javascript
// Combined format (most detailed)
app.use(morgan("combined"));

// Common format
app.use(morgan("common"));

// Dev format (color coded)
app.use(morgan("dev"));

// Short format
app.use(morgan("short"));

// Tiny format (smallest)
app.use(morgan("tiny"));
```

---

## Using Morgan in Express

Basic usage

```javascript
const express = require("express");
const morgan = require("morgan");

const app = express();

// Log all requests
app.use(morgan("dev"));

app.get("/", (req, res) => {
  res.send("Home");
});

app.get("/about", (req, res) => {
  res.send("About");
});

app.post("/users", (req, res) => {
  res.status(201).send("User created");
});

app.listen(3000);
```

Conditional logging

```javascript
// Log only errors (status 400 or higher)
app.use(morgan("dev", {
  skip: (req, res) => res.statusCode < 400
}));

// Log only successful requests
app.use(morgan("dev", {
  skip: (req, res) => res.statusCode >= 400
}));

// Log only POST requests
app.use(morgan("dev", {
  skip: (req, res) => req.method !== "POST"
}));
```

Custom token

```javascript
// Add custom token for user id
morgan.token("userId", (req, res) => {
  return req.user ? req.user.id : "anonymous";
});

// Use custom token
app.use(morgan(":method :url :userId :status - :response-time ms"));
```

---

## Writing Logs to File

Log to file instead of console

```javascript
const express = require("express");
const morgan = require("morgan");
const fs = require("fs");
const path = require("path");

const app = express();

// Create logs folder if it doesn't exist
const logDir = "logs";
if (!fs.existsSync(logDir)) {
  fs.mkdirSync(logDir);
}

// Create write stream for access logs
const accessLogStream = fs.createWriteStream(
  path.join(logDir, "access.log"),
  { flags: "a" } // 'a' means append
);

// Log to file
app.use(morgan("combined", { stream: accessLogStream }));

// Also log to console for development
app.use(morgan("dev"));

app.get("/", (req, res) => {
  res.send("Home");
});

app.listen(3000);
```

Different log files for different environments

```javascript
const env = process.env.NODE_ENV || "development";

if (env === "production") {
  // Production: log to file
  const accessLogStream = fs.createWriteStream(
    path.join(logDir, "access.log"),
    { flags: "a" }
  );
  app.use(morgan("combined", { stream: accessLogStream }));
} else {
  // Development: log to console
  app.use(morgan("dev"));
}
```

---

## What is Winston

Morgan is good for request logging

But for application logging, we need more

Winston is a powerful logging library

Features of Winston

```text
Multiple log levels (error, warn, info, debug)
Log to multiple places (console, file, database)
Custom formats
Log rotation
Very flexible
```

When to use Morgan vs Winston

| Feature | Morgan | Winston |
| ------- | ------ | ------- |
| Request logging | Yes | Can be configured |
| Application logging | No | Yes |
| Log levels | No | Yes |
| Multiple transports | No | Yes |
| Log rotation | No | Yes |

Use both together

```text
Morgan -> HTTP request logs
Winston -> Application, error, debug logs
```

---

## Installing Winston

Install Winston

```bash
npm install winston
```

Create a logger configuration

Create utils/logger.js

```javascript
const winston = require("winston");
const path = require("path");

// Define log format
const logFormat = winston.format.combine(
  winston.format.timestamp(),
  winston.format.errors({ stack: true }),
  winston.format.json()
);

// Create logger
const logger = winston.createLogger({
  level: process.env.LOG_LEVEL || "info",
  format: logFormat,
  transports: [
    // Error logs to separate file
    new winston.transports.File({
      filename: path.join("logs", "error.log"),
      level: "error"
    }),
    // All logs to combined file
    new winston.transports.File({
      filename: path.join("logs", "combined.log")
    })
  ]
});

// In development, also log to console
if (process.env.NODE_ENV !== "production") {
  logger.add(new winston.transports.Console({
    format: winston.format.simple()
  }));
}

module.exports = logger;
```

---

## Log Levels

Winston has six log levels

| Level | Priority | Meaning |
| ----- | -------- | ------- |
| error | 0 | Critical errors, app cannot function |
| warn | 1 | Warning, something suspicious |
| info | 2 | General information |
| http | 3 | HTTP requests |
| verbose | 4 | Detailed information |
| debug | 5 | Debugging information |

Using log levels

```javascript
const logger = require("./utils/logger");

// Error level - something broke
logger.error("Database connection failed");
logger.error("Failed to save user", { userId: 123, error: err.message });

// Warn level - something suspicious
logger.warn("Failed login attempt", { email: "admin@example.com" });
logger.warn("Rate limit exceeded", { ip: "192.168.1.1" });

// Info level - normal operations
logger.info("Server started on port 3000");
logger.info("User registered", { userId: 123, email: "john@example.com" });

// Debug level - detailed info for debugging
logger.debug("Query parameters", { params: req.query });
logger.debug("User object", { user: req.user });
```

You can set minimum log level

```javascript
// Only show error, warn, and info (not debug)
logger.level = "info";

// Show everything including debug
logger.level = "debug";
```

---

## Transports in Winston

Transports are where logs are sent

Console transport

```javascript
logger.add(new winston.transports.Console({
  format: winston.format.simple(),
  level: "info"
}));
```

File transport

```javascript
logger.add(new winston.transports.File({
  filename: "logs/app.log",
  maxsize: 10485760, // 10MB
  maxFiles: 5,
  tailable: true
}));
```

Multiple files for different levels

```javascript
const logger = winston.createLogger({
  transports: [
    // All logs
    new winston.transports.File({ filename: "logs/combined.log" }),
    
    // Only errors
    new winston.transports.File({ 
      filename: "logs/error.log", 
      level: "error" 
    }),
    
    // Only warnings and errors
    new winston.transports.File({ 
      filename: "logs/warn.log", 
      level: "warn" 
    })
  ]
});
```

Daily rotate files

```bash
npm install winston-daily-rotate-file
```

```javascript
const DailyRotateFile = require("winston-daily-rotate-file");

const transport = new DailyRotateFile({
  filename: "logs/application-%DATE%.log",
  datePattern: "YYYY-MM-DD",
  maxSize: "20m",
  maxFiles: "14d"
});

logger.add(transport);
```

---

## Creating a Logger with Winston

Complete logger configuration

Create utils/logger.js

```javascript
const winston = require("winston");
const path = require("path");
const fs = require("fs");

// Ensure logs directory exists
const logDir = "logs";
if (!fs.existsSync(logDir)) {
  fs.mkdirSync(logDir);
}

// Define custom formats
const myFormat = winston.format.printf(({ level, message, timestamp, ...meta }) => {
  let log = `${timestamp} [${level.toUpperCase()}]: ${message}`;
  
  if (Object.keys(meta).length > 0) {
    log += ` ${JSON.stringify(meta)}`;
  }
  
  return log;
});

// Create logger
const logger = winston.createLogger({
  level: process.env.LOG_LEVEL || "info",
  format: winston.format.combine(
    winston.format.timestamp({ format: "YYYY-MM-DD HH:mm:ss" }),
    winston.format.errors({ stack: true }),
    myFormat
  ),
  transports: [
    // Console transport for development
    new winston.transports.Console({
      format: winston.format.combine(
        winston.format.colorize(),
        myFormat
      )
    }),
    
    // File transport for all logs
    new winston.transports.File({
      filename: path.join(logDir, "app.log"),
      maxsize: 10485760, // 10MB
      maxFiles: 5
    }),
    
    // Separate error log file
    new winston.transports.File({
      filename: path.join(logDir, "error.log"),
      level: "error",
      maxsize: 10485760,
      maxFiles: 5
    })
  ]
});

// Create stream for Morgan (to work with Winston)
logger.stream = {
  write: (message) => {
    logger.http(message.trim());
  }
};

module.exports = logger;
```

---

## Complete Logging System

server.js with both Morgan and Winston

```javascript
require("dotenv").config();
const express = require("express");
const morgan = require("morgan");
const logger = require("./utils/logger");

const app = express();

// Morgan with Winston stream
app.use(morgan("combined", { stream: logger.stream }));

// Also log to console in development
if (process.env.NODE_ENV !== "production") {
  app.use(morgan("dev"));
}

app.use(express.json());

// Request logging middleware
app.use((req, res, next) => {
  logger.info(`${req.method} ${req.url} - Request received`);
  next();
});

// Routes
app.get("/", (req, res) => {
  logger.info("Home page accessed");
  res.send("Welcome to Logging Demo");
});

app.get("/students", (req, res) => {
  logger.info("Fetching all students");
  res.json({ students: [] });
});

app.post("/students", (req, res) => {
  const { name, email } = req.body;
  
  logger.info("Creating new student", { name, email });
  
  res.status(201).json({
    success: true,
    message: "Student created"
  });
});

app.get("/students/:id", (req, res) => {
  const id = req.params.id;
  
  logger.debug("Fetching student by id", { id });
  
  if (id === "999") {
    logger.warn("Student not found", { id });
    return res.status(404).json({ error: "Student not found" });
  }
  
  res.json({ id, name: "John Doe" });
});

// Error logging middleware
app.use((err, req, res, next) => {
  logger.error("Unhandled error", {
    error: err.message,
    stack: err.stack,
    url: req.url,
    method: req.method
  });
  
  res.status(500).json({
    success: false,
    message: "Something went wrong"
  });
});

// Start server
const PORT = process.env.PORT || 3000;
app.listen(PORT, () => {
  logger.info(`Server started on port ${PORT}`);
  logger.info(`Environment: ${process.env.NODE_ENV || "development"}`);
});
```

Usage in controllers

```javascript
const logger = require("../utils/logger");

class StudentController {
  async getAllStudents(req, res) {
    try {
      logger.info("Getting all students - started");
      
      const students = await Student.find();
      
      logger.info("Getting all students - completed", {
        count: students.length
      });
      
      res.json(students);
    } catch (error) {
      logger.error("Failed to get students", {
        error: error.message,
        stack: error.stack
      });
      
      res.status(500).json({ error: "Internal server error" });
    }
  }
  
  async createStudent(req, res) {
    const { name, email } = req.body;
    
    logger.info("Creating student", { name, email });
    
    try {
      const student = await Student.create({ name, email });
      
      logger.info("Student created successfully", {
        studentId: student._id
      });
      
      res.status(201).json(student);
    } catch (error) {
      logger.error("Failed to create student", {
        error: error.message,
        name,
        email
      });
      
      res.status(400).json({ error: error.message });
    }
  }
}
```

---

## Practice Exercises

### Exercise 1

Create a logger that writes to three files

```text
logs/error.log (only errors)
logs/warn.log (warnings and errors)
logs/all.log (everything)
```

### Exercise 2

Add request duration logging

Log how long each request takes

### Exercise 3

Create a log rotation system

Keep logs for 7 days only

### Exercise 4

Add database query logging

Log all queries that take more than 1 second

### Exercise 5

Create a security audit log

Log all login attempts (success and failure)

Log all password changes

Log all admin actions

---

## Interview Questions

### What is logging

Logging is recording events and information about application execution

### Why is logging important

For debugging, monitoring, security auditing, and performance analysis

### What is Morgan

Morgan is HTTP request logger middleware for Express

### What is the difference between Morgan and Winston

Morgan is for HTTP request logging
Winston is for general application logging with multiple levels and transports

### What are log levels in Winston

error, warn, info, http, verbose, debug (from highest to lowest priority)

### What is a transport in Winston

A transport is where logs are sent (console, file, database, etc.)

### How do you log errors with stack trace

logger.error("Error message", { stack: err.stack })

### Why should you use different log levels

To filter and prioritize log messages based on importance

---

## Summary

In this session, you learned

* What logging is and why it matters
* Different types of logs (request, error, application, security)
* What Morgan is and how to use it
* Morgan log formats (combined, common, dev, short, tiny)
* Writing logs to files
* What Winston is and why to use it
* Log levels in Winston
* Transports in Winston
* Complete logging system with Morgan and Winston

Good logging makes debugging and monitoring much easier

---