## Table of Contents

* [What is API Documentation](#what-is-api-documentation)
* [Why API Documentation is Important](#why-api-documentation-is-important)
* [What is Swagger](#what-is-swagger)
* [What is OpenAPI](#what-is-openapi)
* [Installing Swagger](#installing-swagger)
* [Basic Swagger Setup](#basic-swagger-setup)
* [Writing API Documentation](#writing-api-documentation)
* [Documenting Routes](#documenting-routes)
* [Documenting Request Body](#documenting-request-body)
* [Documenting Parameters](#documenting-parameters)
* [Documenting Responses](#documenting-responses)
* [Complete Documentation Example](#complete-documentation-example)
* [Practice Exercises](#practice-exercises)
* [Interview Questions](#interview-questions)
* [Summary](#summary)

---

## What is API Documentation

API documentation explains how to use your API

It tells other developers

```text
What endpoints exist
What HTTP methods to use
What parameters are needed
What response to expect
What errors can happen
```

Think of it like a user manual for your API

Without documentation

```text
Developers don't know how to use your API
They have to read your code
They make mistakes
They ask you many questions
They might not use your API at all
```

With documentation

```text
Developers understand your API quickly
They know exactly what to do
They make fewer mistakes
They can integrate faster
More people use your API
```

---

## Why API Documentation is Important

Good documentation benefits everyone

For API creators

```text
Less support questions
Fewer bug reports
Professional appearance
Easier to maintain
```

For API users

```text
Understand API quickly
Know what data to send
Know what to expect back
Easy to test endpoints
Faster development
```

For the company

```text
More developers use the API
Faster integrations
Better product adoption
Higher revenue potential
```

Example of bad documentation

```text
Endpoint: /users
Method: POST
Send some data
Get something back
```

Example of good documentation

```text
Endpoint: POST /api/users
Description: Creates a new user account

Request Body:
{
  "name": "string (required) - User's full name",
  "email": "string (required) - Valid email address",
  "password": "string (required) - Min 6 characters",
  "age": "number (optional) - User's age"
}

Response (201 Created):
{
  "success": true,
  "data": {
    "id": "string",
    "name": "string",
    "email": "string",
    "createdAt": "date"
  }
}

Error Responses:
400 Bad Request - Missing required fields
409 Conflict - Email already exists
500 Server Error - Something went wrong
```

---

## What is Swagger

Swagger is a tool for documenting APIs

It creates interactive documentation

Users can test your API directly from the documentation

Swagger features

```text
Interactive API explorer
Auto-generated documentation
Works with OpenAPI specification
Supports many programming languages
Beautiful user interface
```

What Swagger looks like

```text
Your API endpoints appear in a list
Click on any endpoint
See details about request and response
Click "Try it out"
Enter parameters
Execute request
See real response
```

This is much better than static documentation

---

## What is OpenAPI

OpenAPI is a specification (a set of rules)

It describes how to document REST APIs

OpenAPI uses JSON or YAML format

Swagger is the tool that implements OpenAPI

Think of it like this

```text
OpenAPI = Grammar rules for a language
Swagger = Dictionary and translator that uses those rules
```

OpenAPI specification includes

```text
What endpoints exist
What parameters each endpoint takes
What responses each endpoint returns
Authentication methods
Data models
```

---

## Installing Swagger

Create a new project

```bash
mkdir api-documentation
cd api-documentation
npm init -y
```

Install packages

```bash
npm install express mongoose swagger-ui-express swagger-jsdoc
```

What each package does

```text
express -> Web framework
swagger-ui-express -> Serves Swagger UI interface
swagger-jsdoc -> Converts JSDoc comments to OpenAPI spec
```

Create server.js

---

## Basic Swagger Setup

Basic setup for Swagger documentation

```javascript
const express = require("express");
const swaggerUi = require("swagger-ui-express");
const swaggerJsdoc = require("swagger-jsdoc");

const app = express();

// Swagger definition
const swaggerOptions = {
  definition: {
    openapi: "3.0.0",
    info: {
      title: "Student Management API",
      description: "API for managing students",
      version: "1.0.0",
      contact: {
        name: "API Support",
        email: "support@example.com"
      }
    },
    servers: [
      {
        url: "http://localhost:3000",
        description: "Development server"
      }
    ]
  },
  apis: ["./routes/*.js"] // Path to files with documentation
};

const swaggerDocs = swaggerJsdoc(swaggerOptions);
app.use("/api-docs", swaggerUi.serve, swaggerUi.setup(swaggerDocs));

app.get("/", (req, res) => {
  res.send("Visit /api-docs for API documentation");
});

app.listen(3000, () => {
  console.log("Server running on port 3000");
  console.log("Swagger docs at http://localhost:3000/api-docs");
});
```

Now visit http://localhost:3000/api-docs

You will see the Swagger UI

---

## Writing API Documentation

Documentation is written as comments in your code

Swagger-jsdoc reads these comments

Example comment format

```javascript
/**
 * @swagger
 * /students:
 *   get:
 *     summary: Get all students
 *     description: Returns a list of all students
 *     responses:
 *       200:
 *         description: Success
 */
```

These comments go above your route handlers

---

## Documenting Routes

Document a simple GET route

```javascript
/**
 * @swagger
 * /students:
 *   get:
 *     summary: Get all students
 *     tags: [Students]
 *     responses:
 *       200:
 *         description: List of students
 *         content:
 *           application/json:
 *             schema:
 *               type: array
 *               items:
 *                 $ref: '#/components/schemas/Student'
 */
app.get("/students", (req, res) => {
  res.json(students);
});
```

Document a GET route with parameter

```javascript
/**
 * @swagger
 * /students/{id}:
 *   get:
 *     summary: Get student by ID
 *     tags: [Students]
 *     parameters:
 *       - in: path
 *         name: id
 *         required: true
 *         schema:
 *           type: string
 *         description: Student ID
 *     responses:
 *       200:
 *         description: Student found
 *       404:
 *         description: Student not found
 */
app.get("/students/:id", (req, res) => {
  const student = students.find(s => s.id === req.params.id);
  if (!student) {
    return res.status(404).json({ error: "Student not found" });
  }
  res.json(student);
});
```

Document POST route

```javascript
/**
 * @swagger
 * /students:
 *   post:
 *     summary: Create a new student
 *     tags: [Students]
 *     requestBody:
 *       required: true
 *       content:
 *         application/json:
 *           schema:
 *             type: object
 *             required:
 *               - name
 *               - email
 *             properties:
 *               name:
 *                 type: string
 *                 description: Student name
 *               email:
 *                 type: string
 *                 description: Student email
 *               age:
 *                 type: number
 *                 description: Student age
 *     responses:
 *       201:
 *         description: Student created
 *       400:
 *         description: Invalid input
 */
app.post("/students", (req, res) => {
  const { name, email, age } = req.body;
  const newStudent = { id: Date.now(), name, email, age };
  students.push(newStudent);
  res.status(201).json(newStudent);
});
```

---

## Documenting Request Body

For POST and PUT requests, document the body

```javascript
/**
 * @swagger
 * /students:
 *   post:
 *     summary: Create student
 *     requestBody:
 *       required: true
 *       content:
 *         application/json:
 *           schema:
 *             type: object
 *             properties:
 *               name:
 *                 type: string
 *                 example: "John Doe"
 *               email:
 *                 type: string
 *                 format: email
 *                 example: "john@example.com"
 *               age:
 *                 type: integer
 *                 minimum: 18
 *                 maximum: 60
 *                 example: 25
 *               course:
 *                 type: string
 *                 enum: ["Computer Science", "Mathematics", "Physics"]
 *                 example: "Computer Science"
 *     responses:
 *       201:
 *         description: Created
 */
```

---

## Documenting Parameters

Parameters can be in different places

Path parameters

```javascript
/**
 * @swagger
 * /students/{id}:
 *   get:
 *     parameters:
 *       - in: path
 *         name: id
 *         required: true
 *         schema:
 *           type: string
 *         description: The student ID
 */
```

Query parameters

```javascript
/**
 * @swagger
 * /students:
 *   get:
 *     parameters:
 *       - in: query
 *         name: page
 *         schema:
 *           type: integer
 *           default: 1
 *         description: Page number for pagination
 *       - in: query
 *         name: limit
 *         schema:
 *           type: integer
 *           default: 10
 *         description: Number of items per page
 *       - in: query
 *         name: course
 *         schema:
 *           type: string
 *         description: Filter by course name
 */
```

Header parameters

```javascript
/**
 * @swagger
 * /students:
 *   get:
 *     parameters:
 *       - in: header
 *         name: Authorization
 *         required: true
 *         schema:
 *           type: string
 *         description: Bearer token for authentication
 */
```

---

## Documenting Responses

Document different response status codes

```javascript
/**
 * @swagger
 * /students/{id}:
 *   get:
 *     responses:
 *       200:
 *         description: Success
 *         content:
 *           application/json:
 *             schema:
 *               type: object
 *               properties:
 *                 success:
 *                   type: boolean
 *                 data:
 *                   $ref: '#/components/schemas/Student'
 *       400:
 *         description: Invalid ID format
 *         content:
 *           application/json:
 *             schema:
 *               type: object
 *               properties:
 *                 success:
 *                   type: boolean
 *                 message:
 *                   type: string
 *       404:
 *         description: Student not found
 *       500:
 *         description: Server error
 */
```

---

## Creating Schemas/Models

Define reusable schemas

```javascript
/**
 * @swagger
 * components:
 *   schemas:
 *     Student:
 *       type: object
 *       required:
 *         - name
 *         - email
 *       properties:
 *         id:
 *           type: string
 *           description: Auto-generated ID
 *         name:
 *           type: string
 *           description: Student name
 *         email:
 *           type: string
 *           format: email
 *           description: Student email
 *         age:
 *           type: integer
 *           minimum: 18
 *           maximum: 60
 *           description: Student age
 *         course:
 *           type: string
 *           enum: [Computer Science, Mathematics, Physics]
 *           description: Enrolled course
 *         createdAt:
 *           type: string
 *           format: date-time
 *           description: Creation timestamp
 *     
 *     Error:
 *       type: object
 *       properties:
 *         success:
 *           type: boolean
 *           example: false
 *         message:
 *           type: string
 *     
 *     Success:
 *       type: object
 *       properties:
 *         success:
 *           type: boolean
 *           example: true
 *         data:
 *           type: object
 */
```

Now you can reference these schemas

```javascript
/**
 * @swagger
 * /students:
 *   get:
 *     responses:
 *       200:
 *         description: Success
 *         content:
 *           application/json:
 *             schema:
 *               type: object
 *               properties:
 *                 success:
 *                   type: boolean
 *                 data:
 *                   type: array
 *                   items:
 *                     $ref: '#/components/schemas/Student'
 */
```

---

## Complete Documentation Example

Project structure

```text
api-documentation/
├── server.js
├── routes/
│   └── studentRoutes.js
└── models/
    └── Student.js
```

server.js

```javascript
const express = require("express");
const swaggerUi = require("swagger-ui-express");
const swaggerJsdoc = require("swagger-jsdoc");
const studentRoutes = require("./routes/studentRoutes");

const app = express();
app.use(express.json());

// Swagger configuration
const swaggerOptions = {
  definition: {
    openapi: "3.0.0",
    info: {
      title: "Student Management API",
      version: "1.0.0",
      description: "Complete API for managing students",
      contact: {
        name: "API Support",
        email: "support@example.com"
      },
      license: {
        name: "MIT",
        url: "https://opensource.org/licenses/MIT"
      }
    },
    servers: [
      {
        url: "http://localhost:3000/api/v1",
        description: "Development server"
      }
    ],
    components: {
      securitySchemes: {
        bearerAuth: {
          type: "http",
          scheme: "bearer",
          bearerFormat: "JWT"
        }
      }
    },
    security: [
      {
        bearerAuth: []
      }
    ]
  },
  apis: ["./routes/*.js"]
};

const swaggerDocs = swaggerJsdoc(swaggerOptions);
app.use("/api-docs", swaggerUi.serve, swaggerUi.setup(swaggerDocs));

// Routes
app.use("/api/v1", studentRoutes);

app.listen(3000, () => {
  console.log("Server running on port 3000");
  console.log("Swagger UI: http://localhost:3000/api-docs");
});
```

routes/studentRoutes.js

```javascript
const express = require("express");
const router = express.Router();

let students = [
  { id: 1, name: "John Doe", email: "john@example.com", age: 20 }
];

/**
 * @swagger
 * /students:
 *   get:
 *     summary: Get all students
 *     tags: [Students]
 *     security: []
 *     responses:
 *       200:
 *         description: List of students
 *         content:
 *           application/json:
 *             schema:
 *               type: object
 *               properties:
 *                 success:
 *                   type: boolean
 *                 count:
 *                   type: integer
 *                 data:
 *                   type: array
 *                   items:
 *                     $ref: '#/components/schemas/Student'
 */
router.get("/students", (req, res) => {
  res.json({
    success: true,
    count: students.length,
    data: students
  });
});

/**
 * @swagger
 * /students/{id}:
 *   get:
 *     summary: Get student by ID
 *     tags: [Students]
 *     security: []
 *     parameters:
 *       - in: path
 *         name: id
 *         required: true
 *         schema:
 *           type: integer
 *         description: Student ID
 *     responses:
 *       200:
 *         description: Student found
 *         content:
 *           application/json:
 *             schema:
 *               type: object
 *               properties:
 *                 success:
 *                   type: boolean
 *                 data:
 *                   $ref: '#/components/schemas/Student'
 *       404:
 *         description: Student not found
 */
router.get("/students/:id", (req, res) => {
  const student = students.find(s => s.id === parseInt(req.params.id));
  
  if (!student) {
    return res.status(404).json({
      success: false,
      message: "Student not found"
    });
  }
  
  res.json({
    success: true,
    data: student
  });
});

/**
 * @swagger
 * /students:
 *   post:
 *     summary: Create a new student
 *     tags: [Students]
 *     security: []
 *     requestBody:
 *       required: true
 *       content:
 *         application/json:
 *           schema:
 *             type: object
 *             required:
 *               - name
 *               - email
 *             properties:
 *               name:
 *                 type: string
 *                 example: "Jane Smith"
 *               email:
 *                 type: string
 *                 format: email
 *                 example: "jane@example.com"
 *               age:
 *                 type: integer
 *                 example: 22
 *     responses:
 *       201:
 *         description: Student created
 *       400:
 *         description: Validation error
 */
router.post("/students", (req, res) => {
  const { name, email, age } = req.body;
  
  if (!name || !email) {
    return res.status(400).json({
      success: false,
      message: "Name and email are required"
    });
  }
  
  const newStudent = {
    id: students.length + 1,
    name,
    email,
    age: age || null
  };
  
  students.push(newStudent);
  
  res.status(201).json({
    success: true,
    data: newStudent
  });
});

/**
 * @swagger
 * /students/{id}:
 *   put:
 *     summary: Update a student
 *     tags: [Students]
 *     security: []
 *     parameters:
 *       - in: path
 *         name: id
 *         required: true
 *         schema:
 *           type: integer
 *     requestBody:
 *       required: true
 *       content:
 *         application/json:
 *           schema:
 *             type: object
 *             properties:
 *               name:
 *                 type: string
 *               email:
 *                 type: string
 *               age:
 *                 type: integer
 *     responses:
 *       200:
 *         description: Student updated
 *       404:
 *         description: Student not found
 */
router.put("/students/:id", (req, res) => {
  const id = parseInt(req.params.id);
  const index = students.findIndex(s => s.id === id);
  
  if (index === -1) {
    return res.status(404).json({
      success: false,
      message: "Student not found"
    });
  }
  
  students[index] = { ...students[index], ...req.body };
  
  res.json({
    success: true,
    data: students[index]
  });
});

/**
 * @swagger
 * /students/{id}:
 *   delete:
 *     summary: Delete a student
 *     tags: [Students]
 *     security: []
 *     parameters:
 *       - in: path
 *         name: id
 *         required: true
 *         schema:
 *           type: integer
 *     responses:
 *       200:
 *         description: Student deleted
 *       404:
 *         description: Student not found
 */
router.delete("/students/:id", (req, res) => {
  const id = parseInt(req.params.id);
  const index = students.findIndex(s => s.id === id);
  
  if (index === -1) {
    return res.status(404).json({
      success: false,
      message: "Student not found"
    });
  }
  
  students.splice(index, 1);
  
  res.json({
    success: true,
    message: "Student deleted successfully"
  });
});

module.exports = router;
```

---

## Practice Exercises

### Exercise 1

Document a products API

Include endpoints for GET, POST, PUT, DELETE

### Exercise 2

Add authentication to your documentation

Use bearerAuth security scheme

### Exercise 3

Document error responses

Include 400, 401, 403, 404, 500

### Exercise 4

Add example values for all parameters

### Exercise 5

Create tags to group endpoints

Example: Students, Products, Orders, Users

---

## Interview Questions

### What is API documentation

API documentation explains how to use an API including endpoints, parameters, and responses

### What is Swagger

Swagger is a tool for creating interactive API documentation

### What is OpenAPI

OpenAPI is a specification for describing REST APIs

### What is the difference between Swagger and OpenAPI

OpenAPI is the specification
Swagger is the tool that implements the specification

### What is swagger-ui-express

Middleware that serves Swagger UI interface in Express applications

### What is swagger-jsdoc

Library that converts JSDoc comments to OpenAPI specification

### Why is API documentation important

It helps developers understand and use your API correctly and efficiently

---

## Summary

In this session, you learned

* What API documentation is
* Why API documentation is important
* What Swagger and OpenAPI are
* How to install and configure Swagger
* How to document routes, parameters, and request body
* How to document responses
* How to create reusable schemas
* How to group endpoints with tags

Good documentation makes your API professional and easy to use

---