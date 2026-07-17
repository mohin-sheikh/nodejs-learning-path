## Table of Contents

* [What is a REST API](#what-is-a-rest-api)
* [REST API Rules](#rest-api-rules)
* [Setting Up the Project](#setting-up-the-project)
* [Complete Student API](#complete-student-api)
* [GET All Students](#get-all-students)
* [GET Single Student](#get-single-student)
* [POST Create Student](#post-create-student)
* [PUT Update Student](#put-update-student)
* [DELETE Remove Student](#delete-remove-student)
* [Adding Validation](#adding-validation)
* [Proper Status Codes](#proper-status-codes)
* [Error Handling](#error-handling)
* [Complete Code with Error Handling](#complete-code-with-error-handling)
* [Testing the API](#testing-the-api)
* [Practice Exercises](#practice-exercises)
* [Interview Questions](#interview-questions)
* [Summary](#summary)

---

## What is a REST API

REST API is a way to build web services

REST stands for Representational State Transfer

It is not a tool or library

It is a set of rules that most backend applications follow

Think of REST API like a restaurant menu

```text
Customer looks at menu -> Knows what to order
Developer looks at API -> Knows what endpoints exist
```

A good REST API is predictable

If you know how one REST API works, you know how most work

---

## REST API Rules

Rule 1 - Use proper HTTP methods

```text
GET    -> Get data
POST   -> Create new data
PUT    -> Update existing data
DELETE -> Remove data
```

Rule 2 - Use proper status codes

```text
200 -> Success
201 -> Created
400 -> Bad request
404 -> Not found
500 -> Server error
```

Rule 3 - Use nouns for endpoints, not verbs

```text
Good -> /students
Bad  -> /getAllStudents

Good -> /products
Bad  -> /fetchProducts
```

Rule 4 - Use plural names

```text
Good -> /students
Bad  -> /student

Good -> /products
Bad  -> /product
```

Rule 5 - Use nested resources for related data

```text
/students/5/courses
/students/5/courses/2
```

---

## Setting Up the Project

Create a new project

```bash
mkdir rest-api-express
cd rest-api-express
npm init -y
npm install express
```

Create a file named server.js

Our data will be stored in an array

```javascript
const express = require("express");
const app = express();

app.use(express.json());

let students = [
  {
    id: 1,
    name: "John Doe",
    age: 20,
    course: "Computer Science",
    email: "john@example.com"
  },
  {
    id: 2,
    name: "Jane Smith",
    age: 22,
    course: "Mathematics",
    email: "jane@example.com"
  },
  {
    id: 3,
    name: "Mike Johnson",
    age: 21,
    course: "Physics",
    email: "mike@example.com"
  }
];

app.listen(3000, () => {
  console.log("REST API running on port 3000");
});
```

---

## GET All Students

This endpoint returns all students

```javascript
app.get("/students", (req, res) => {
  res.status(200).json({
    success: true,
    count: students.length,
    data: students
  });
});
```

Response when you visit /students

```json
{
  "success": true,
  "count": 3,
  "data": [
    {
      "id": 1,
      "name": "John Doe",
      "age": 20,
      "course": "Computer Science",
      "email": "john@example.com"
    }
  ]
}
```

Why we add success and count

```text
success -> Tells client if request worked
count -> Tells client how many items returned
data -> The actual data
```

This is a common pattern in REST APIs

---

## GET Single Student

This endpoint returns one student by id

```javascript
app.get("/students/:id", (req, res) => {
  const id = parseInt(req.params.id);
  const student = students.find(s => s.id === id);
  
  if (student) {
    res.status(200).json({
      success: true,
      data: student
    });
  } else {
    res.status(404).json({
      success: false,
      message: `Student with id ${id} not found`
    });
  }
});
```

When student exists, visit /students/1

```json
{
  "success": true,
  "data": {
    "id": 1,
    "name": "John Doe",
    "age": 20,
    "course": "Computer Science",
    "email": "john@example.com"
  }
}
```

When student does not exist, visit /students/999

```json
{
  "success": false,
  "message": "Student with id 999 not found"
}
```

---

## POST Create Student

This endpoint creates a new student

```javascript
app.post("/students", (req, res) => {
  const { name, age, course, email } = req.body;
  
  const newStudent = {
    id: students.length + 1,
    name: name,
    age: age,
    course: course,
    email: email
  };
  
  students.push(newStudent);
  
  res.status(201).json({
    success: true,
    data: newStudent
  });
});
```

Send a POST request to /students with this body

```json
{
  "name": "Sarah Wilson",
  "age": 23,
  "course": "Biology",
  "email": "sarah@example.com"
}
```

Response

```json
{
  "success": true,
  "data": {
    "id": 4,
    "name": "Sarah Wilson",
    "age": 23,
    "course": "Biology",
    "email": "sarah@example.com"
  }
}
```

Notice we use status 201 for created

---

## PUT Update Student

This endpoint updates an existing student

```javascript
app.put("/students/:id", (req, res) => {
  const id = parseInt(req.params.id);
  const { name, age, course, email } = req.body;
  
  const index = students.findIndex(s => s.id === id);
  
  if (index !== -1) {
    students[index] = {
      ...students[index],
      name: name || students[index].name,
      age: age || students[index].age,
      course: course || students[index].course,
      email: email || students[index].email
    };
    
    res.status(200).json({
      success: true,
      data: students[index]
    });
  } else {
    res.status(404).json({
      success: false,
      message: `Student with id ${id} not found`
    });
  }
});
```

Send a PUT request to /students/1 with this body

```json
{
  "name": "John Updated",
  "age": 21
}
```

Response

```json
{
  "success": true,
  "data": {
    "id": 1,
    "name": "John Updated",
    "age": 21,
    "course": "Computer Science",
    "email": "john@example.com"
  }
}
```

Notice we only update the fields sent in the request

The other fields stay the same

---

## DELETE Remove Student

This endpoint removes a student

```javascript
app.delete("/students/:id", (req, res) => {
  const id = parseInt(req.params.id);
  
  const index = students.findIndex(s => s.id === id);
  
  if (index !== -1) {
    students.splice(index, 1);
    
    res.status(200).json({
      success: true,
      message: `Student with id ${id} deleted successfully`
    });
  } else {
    res.status(404).json({
      success: false,
      message: `Student with id ${id} not found`
    });
  }
});
```

Send a DELETE request to /students/1

Response

```json
{
  "success": true,
  "message": "Student with id 1 deleted successfully"
}
```

---

## Adding Validation

We should validate data before saving

Add validation to POST request

```javascript
app.post("/students", (req, res) => {
  const { name, age, course, email } = req.body;
  
  // Check if name exists
  if (!name) {
    return res.status(400).json({
      success: false,
      message: "Name is required"
    });
  }
  
  // Check if age is valid
  if (!age) {
    return res.status(400).json({
      success: false,
      message: "Age is required"
    });
  }
  
  if (age < 18 || age > 60) {
    return res.status(400).json({
      success: false,
      message: "Age must be between 18 and 60"
    });
  }
  
  // Check if email exists
  if (!email) {
    return res.status(400).json({
      success: false,
      message: "Email is required"
    });
  }
  
  // Check if email already exists
  const existingStudent = students.find(s => s.email === email);
  
  if (existingStudent) {
    return res.status(400).json({
      success: false,
      message: "Email already exists"
    });
  }
  
  const newStudent = {
    id: students.length + 1,
    name: name,
    age: age,
    course: course || "Not specified",
    email: email
  };
  
  students.push(newStudent);
  
  res.status(201).json({
    success: true,
    data: newStudent
  });
});
```

Now if someone forgets to send name

Response

```json
{
  "success": false,
  "message": "Name is required"
}
```

---

## Proper Status Codes

Always use the correct status code

| Status Code | Meaning | When to use |
| ----------- | ------- | ----------- |
| 200 | OK | GET, PUT, DELETE successful |
| 201 | Created | POST successful |
| 400 | Bad Request | Validation failed, missing fields |
| 401 | Unauthorized | Not logged in |
| 403 | Forbidden | Logged in but no permission |
| 404 | Not Found | Resource does not exist |
| 500 | Server Error | Something broke on server |

Example usage

```javascript
// Success
res.status(200).json({ success: true });

// Created
res.status(201).json({ success: true });

// Bad request
res.status(400).json({ success: false, message: "Invalid data" });

// Not found
res.status(404).json({ success: false, message: "Not found" });

// Server error
res.status(500).json({ success: false, message: "Server error" });
```

---

## Error Handling

Sometimes things go wrong

We should handle errors properly

Create a global error handler

```javascript
// This catches all errors
app.use((err, req, res, next) => {
  console.error(err.stack);
  
  res.status(500).json({
    success: false,
    message: "Something went wrong on the server"
  });
});
```

For routes that might have errors, use try-catch

```javascript
app.get("/students", async (req, res, next) => {
  try {
    // Some code that might fail
    res.json(students);
  } catch (error) {
    next(error); // Send error to global error handler
  }
});
```

---

## Complete Code with Error Handling

```javascript
const express = require("express");
const app = express();

app.use(express.json());

let students = [
  {
    id: 1,
    name: "John Doe",
    age: 20,
    course: "Computer Science",
    email: "john@example.com"
  },
  {
    id: 2,
    name: "Jane Smith",
    age: 22,
    course: "Mathematics",
    email: "jane@example.com"
  },
  {
    id: 3,
    name: "Mike Johnson",
    age: 21,
    course: "Physics",
    email: "mike@example.com"
  }
];

// GET all students
app.get("/students", (req, res) => {
  res.status(200).json({
    success: true,
    count: students.length,
    data: students
  });
});

// GET single student
app.get("/students/:id", (req, res) => {
  const id = parseInt(req.params.id);
  const student = students.find(s => s.id === id);
  
  if (student) {
    res.status(200).json({
      success: true,
      data: student
    });
  } else {
    res.status(404).json({
      success: false,
      message: `Student with id ${id} not found`
    });
  }
});

// POST create student
app.post("/students", (req, res) => {
  const { name, age, course, email } = req.body;
  
  if (!name) {
    return res.status(400).json({
      success: false,
      message: "Name is required"
    });
  }
  
  if (!age) {
    return res.status(400).json({
      success: false,
      message: "Age is required"
    });
  }
  
  if (age < 18 || age > 60) {
    return res.status(400).json({
      success: false,
      message: "Age must be between 18 and 60"
    });
  }
  
  if (!email) {
    return res.status(400).json({
      success: false,
      message: "Email is required"
    });
  }
  
  const existingStudent = students.find(s => s.email === email);
  
  if (existingStudent) {
    return res.status(400).json({
      success: false,
      message: "Email already exists"
    });
  }
  
  const newStudent = {
    id: students.length + 1,
    name,
    age,
    course: course || "Not specified",
    email
  };
  
  students.push(newStudent);
  
  res.status(201).json({
    success: true,
    data: newStudent
  });
});

// PUT update student
app.put("/students/:id", (req, res) => {
  const id = parseInt(req.params.id);
  const { name, age, course, email } = req.body;
  
  const index = students.findIndex(s => s.id === id);
  
  if (index !== -1) {
    students[index] = {
      ...students[index],
      name: name || students[index].name,
      age: age || students[index].age,
      course: course || students[index].course,
      email: email || students[index].email
    };
    
    res.status(200).json({
      success: true,
      data: students[index]
    });
  } else {
    res.status(404).json({
      success: false,
      message: `Student with id ${id} not found`
    });
  }
});

// DELETE student
app.delete("/students/:id", (req, res) => {
  const id = parseInt(req.params.id);
  
  const index = students.findIndex(s => s.id === id);
  
  if (index !== -1) {
    students.splice(index, 1);
    
    res.status(200).json({
      success: true,
      message: `Student with id ${id} deleted successfully`
    });
  } else {
    res.status(404).json({
      success: false,
      message: `Student with id ${id} not found`
    });
  }
});

// Handle 404 for unknown routes
app.use((req, res) => {
  res.status(404).json({
    success: false,
    message: `Route ${req.url} not found`
  });
});

// Global error handler
app.use((err, req, res, next) => {
  console.error(err.stack);
  
  res.status(500).json({
    success: false,
    message: "Something went wrong on the server"
  });
});

app.listen(3000, () => {
  console.log("REST API running on port 3000");
  console.log("Try these endpoints");
  console.log("GET    http://localhost:3000/students");
  console.log("GET    http://localhost:3000/students/1");
  console.log("POST   http://localhost:3000/students");
  console.log("PUT    http://localhost:3000/students/1");
  console.log("DELETE http://localhost:3000/students/1");
});
```

---

## Testing the API

Start the server

```bash
node server.js
```

Test with curl commands

GET all students

```bash
curl http://localhost:3000/students
```

GET one student

```bash
curl http://localhost:3000/students/1
```

POST create student

```bash
curl -X POST http://localhost:3000/students \
  -H "Content-Type: application/json" \
  -d '{"name":"Sarah","age":23,"course":"Biology","email":"sarah@example.com"}'
```

PUT update student

```bash
curl -X PUT http://localhost:3000/students/1 \
  -H "Content-Type: application/json" \
  -d '{"name":"John Updated"}'
```

DELETE student

```bash
curl -X DELETE http://localhost:3000/students/1
```

Test validation - missing name

```bash
curl -X POST http://localhost:3000/students \
  -H "Content-Type: application/json" \
  -d '{"age":23}'
```

You will get

```json
{
  "success": false,
  "message": "Name is required"
}
```

---

## Practice Exercises

### Exercise 1

Create a products API with validation

Each product should have

```text
id
name
price
category
```

Add validation

```text
Name is required
Price must be greater than 0
Category is required
```

### Exercise 2

Add a filter feature to GET /products

```text
GET /products?category=electronics
GET /products?minPrice=100
GET /products?maxPrice=500
```

### Exercise 3

Add a search feature

```text
GET /students?search=John
```

Return students whose name contains the search term

### Exercise 4

Add pagination

```text
GET /students?page=1&limit=5
```

Return only the requested page of results

### Exercise 5

Add a PATCH endpoint

PATCH is like PUT but for partial updates

Compare PUT and PATCH

---

## Interview Questions

### What does REST stand for

Representational State Transfer

### What are the main HTTP methods in REST API

GET, POST, PUT, DELETE

### What status code is returned for a successful GET request

200 OK

### What status code is returned for a successful POST request

201 Created

### What is the difference between PUT and PATCH

PUT replaces the entire resource
PATCH updates only the fields sent

### Why should we use plural names for endpoints

It is a REST convention and makes the API predictable

Example: /students instead of /student

### What should you return when a resource is not found

404 status code with a message explaining the resource was not found

### Why is validation important

To prevent invalid data from being saved to the database

---

## Summary

In this session, you learned

* What a REST API is
* REST API rules and conventions
* How to build a complete REST API
* GET, POST, PUT, DELETE endpoints
* Proper status codes for each operation
* Adding validation to requests
* Error handling in Express
* Global error handler
* Testing REST APIs with curl

You have built a production-ready REST API using Express

---