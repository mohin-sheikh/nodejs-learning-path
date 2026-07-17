## Table of Contents

* [What is API Testing](#what-is-api-testing)
* [Why Testing is Important](#why-testing-is-important)
* [Types of Tests](#types-of-tests)
* [What is Jest](#what-is-jest)
* [What is Supertest](#what-is-supertest)
* [Installing Jest and Supertest](#installing-jest-and-supertest)
* [Setting Up Test Environment](#setting-up-test-environment)
* [Writing Your First Test](#writing-your-first-test)
* [Testing GET Requests](#testing-get-requests)
* [Testing POST Requests](#testing-post-requests)
* [Testing PUT Requests](#testing-put-requests)
* [Testing DELETE Requests](#testing-delete-requests)
* [Testing Error Responses](#testing-error-responses)
* [Before and After Hooks](#before-and-after-hooks)
* [Complete Testing Example](#complete-testing-example)
* [Practice Exercises](#practice-exercises)
* [Interview Questions](#interview-questions)
* [Summary](#summary)

---

## What is API Testing

API testing is checking if your API works correctly

You write code that calls your API and checks the responses

Think of it like a robot tester

```text
Robot sends request to your API
Robot checks if response is correct
Robot reports if something is wrong
Robot does this every time you change code
```

Example of a test

```text
Test: Create a new student
Step 1: Send POST request to /students
Step 2: Check if status code is 201
Step 3: Check if response has id
Step 4: Check if student exists in database
Result: Test passes or fails
```

---

## Why Testing is Important

Without tests

```text
You change one line of code
Something else breaks
You don't know about it
User finds the bug
You lose trust
```

With tests

```text
You change one line of code
Run tests automatically
Tests show what broke
You fix it before users see it
Confidence to change code
```

Benefits of testing

```text
Find bugs early
Save time and money
Confidence to refactor
Better code quality
Documentation of expected behavior
Prevents regression (breaking old features)
```

---

## Types of Tests

Unit Tests

```text
Test one small piece of code
Example: Test a function that adds two numbers
Fast to run
Many of them
```

Integration Tests

```text
Test how pieces work together
Example: Test database connection with model
Slower than unit tests
Fewer of them
```

API Tests (Endpoint Tests)

```text
Test entire API endpoints
Example: Test POST /students endpoint
Slowest to run
Fewest of them
```

Test Pyramid

```text
        /\
       /  \
      /    \
     / API  \
    /--------\
   / Integration \
  /--------------\
 /     Unit       \
/__________________\
```

Most tests should be unit tests
Fewer integration tests
Even fewer API tests

For beginners, focus on API tests first

---

## What is Jest

Jest is a testing framework from Facebook

It is very popular for Node.js testing

Jest features

```text
Simple syntax
Built-in assertions
Test runner
Mocking support
Code coverage
Watch mode
```

Jest syntax example

```javascript
test("adds 1 + 2 to equal 3", () => {
  expect(1 + 2).toBe(3);
});
```

---

## What is Supertest

Supertest is for testing HTTP endpoints

It lets you send requests to your Express app

Supertest features

```text
Send GET, POST, PUT, DELETE requests
Check status codes
Check response bodies
Works with Jest
No need to run server separately
```

Supertest syntax example

```javascript
request(app)
  .get("/students")
  .expect(200)
  .expect(res => {
    expect(res.body).toHaveProperty("data");
  });
```

---

## Installing Jest and Supertest

Create a new project

```bash
mkdir api-testing
cd api-testing
npm init -y
```

Install production packages

```bash
npm install express mongoose dotenv
```

Install testing packages

```bash
npm install --save-dev jest supertest
```

Update package.json

```json
{
  "scripts": {
    "test": "jest",
    "test:watch": "jest --watch",
    "test:coverage": "jest --coverage"
  }
}
```

Create a folder for tests

```bash
mkdir tests
```

---

## Setting Up Test Environment

Create a separate database for testing

Create .env.test file

```text
NODE_ENV=test
PORT=5001
MONGODB_URI=mongodb+srv://username:password@cluster0.abc123.mongodb.net/
DB_NAME=student_api_test
```

Create test setup file

tests/setup.js

```javascript
const mongoose = require("mongoose");
const { MongoMemoryServer } = require("mongodb-memory-server");

let mongoServer;

// Connect to in-memory database before tests
beforeAll(async () => {
  mongoServer = await MongoMemoryServer.create();
  const uri = mongoServer.getUri();
  await mongoose.connect(uri);
});

// Clear database between tests
afterEach(async () => {
  const collections = mongoose.connection.collections;
  for (const key in collections) {
    await collections[key].deleteMany();
  }
});

// Disconnect after all tests
afterAll(async () => {
  await mongoose.disconnect();
  await mongoServer.stop();
});
```

Install in-memory MongoDB for testing

```bash
npm install --save-dev mongodb-memory-server
```

---

## Writing Your First Test

Create a simple Express app for testing

server.js

```javascript
const express = require("express");
const app = express();

app.use(express.json());

app.get("/", (req, res) => {
  res.json({ message: "Hello World" });
});

app.get("/health", (req, res) => {
  res.status(200).json({ status: "OK" });
});

module.exports = app;
```

Create tests/app.test.js

```javascript
const request = require("supertest");
const app = require("../server");

describe("App Tests", () => {
  test("GET / should return hello message", async () => {
    const response = await request(app)
      .get("/")
      .expect(200);
    
    expect(response.body.message).toBe("Hello World");
  });
  
  test("GET /health should return OK status", async () => {
    const response = await request(app)
      .get("/health")
      .expect(200);
    
    expect(response.body.status).toBe("OK");
  });
});
```

Run tests

```bash
npm test
```

---

## Testing GET Requests

Create a student API to test

models/Student.js

```javascript
const mongoose = require("mongoose");

const studentSchema = new mongoose.Schema({
  name: { type: String, required: true },
  email: { type: String, required: true, unique: true },
  age: { type: Number, min: 18, max: 60 },
  course: { type: String }
});

module.exports = mongoose.model("Student", studentSchema);
```

routes/studentRoutes.js

```javascript
const express = require("express");
const router = express.Router();
const Student = require("../models/Student");

// GET all students
router.get("/", async (req, res) => {
  try {
    const students = await Student.find();
    res.json({
      success: true,
      count: students.length,
      data: students
    });
  } catch (error) {
    res.status(500).json({ success: false, message: error.message });
  }
});

// GET single student
router.get("/:id", async (req, res) => {
  try {
    const student = await Student.findById(req.params.id);
    
    if (!student) {
      return res.status(404).json({ success: false, message: "Student not found" });
    }
    
    res.json({ success: true, data: student });
  } catch (error) {
    res.status(500).json({ success: false, message: error.message });
  }
});

module.exports = router;
```

Tests for GET requests

tests/student.test.js

```javascript
const request = require("supertest");
const app = require("../server");
const Student = require("../models/Student");

describe("Student API Tests", () => {
  describe("GET /api/students", () => {
    test("should return empty array when no students exist", async () => {
      const response = await request(app)
        .get("/api/students")
        .expect(200);
      
      expect(response.body.success).toBe(true);
      expect(response.body.count).toBe(0);
      expect(response.body.data).toEqual([]);
    });
    
    test("should return all students when they exist", async () => {
      // Create test data
      await Student.create([
        { name: "John Doe", email: "john@test.com", age: 20 },
        { name: "Jane Doe", email: "jane@test.com", age: 22 }
      ]);
      
      const response = await request(app)
        .get("/api/students")
        .expect(200);
      
      expect(response.body.success).toBe(true);
      expect(response.body.count).toBe(2);
      expect(response.body.data.length).toBe(2);
      expect(response.body.data[0].name).toBe("John Doe");
    });
  });
  
  describe("GET /api/students/:id", () => {
    test("should return student when valid id provided", async () => {
      const student = await Student.create({
        name: "Test User",
        email: "test@test.com",
        age: 25
      });
      
      const response = await request(app)
        .get(`/api/students/${student._id}`)
        .expect(200);
      
      expect(response.body.success).toBe(true);
      expect(response.body.data.name).toBe("Test User");
      expect(response.body.data.email).toBe("test@test.com");
    });
    
    test("should return 404 when student not found", async () => {
      const fakeId = "507f1f77bcf86cd799439011";
      
      const response = await request(app)
        .get(`/api/students/${fakeId}`)
        .expect(404);
      
      expect(response.body.success).toBe(false);
      expect(response.body.message).toBe("Student not found");
    });
  });
});
```

---

## Testing POST Requests

Add POST route to studentRoutes.js

```javascript
// POST create student
router.post("/", async (req, res) => {
  try {
    const { name, email, age, course } = req.body;
    
    // Validation
    if (!name || !email) {
      return res.status(400).json({
        success: false,
        message: "Name and email are required"
      });
    }
    
    const student = await Student.create({ name, email, age, course });
    
    res.status(201).json({
      success: true,
      data: student
    });
  } catch (error) {
    if (error.code === 11000) {
      return res.status(400).json({
        success: false,
        message: "Email already exists"
      });
    }
    res.status(400).json({ success: false, message: error.message });
  }
});
```

Tests for POST requests

```javascript
describe("POST /api/students", () => {
  test("should create new student with valid data", async () => {
    const newStudent = {
      name: "New Student",
      email: "new@test.com",
      age: 23,
      course: "Computer Science"
    };
    
    const response = await request(app)
      .post("/api/students")
      .send(newStudent)
      .expect(201);
    
    expect(response.body.success).toBe(true);
    expect(response.body.data.name).toBe("New Student");
    expect(response.body.data.email).toBe("new@test.com");
    
    // Verify student exists in database
    const studentInDb = await Student.findOne({ email: "new@test.com" });
    expect(studentInDb).not.toBeNull();
  });
  
  test("should return 400 when name is missing", async () => {
    const invalidStudent = {
      email: "missingname@test.com",
      age: 23
    };
    
    const response = await request(app)
      .post("/api/students")
      .send(invalidStudent)
      .expect(400);
    
    expect(response.body.success).toBe(false);
    expect(response.body.message).toBe("Name and email are required");
  });
  
  test("should return 400 when email already exists", async () => {
    // Create a student first
    await Student.create({
      name: "Existing User",
      email: "existing@test.com",
      age: 20
    });
    
    // Try to create another with same email
    const duplicateStudent = {
      name: "Another User",
      email: "existing@test.com",
      age: 22
    };
    
    const response = await request(app)
      .post("/api/students")
      .send(duplicateStudent)
      .expect(400);
    
    expect(response.body.success).toBe(false);
    expect(response.body.message).toBe("Email already exists");
  });
});
```

---

## Testing PUT Requests

Add PUT route to studentRoutes.js

```javascript
// PUT update student
router.put("/:id", async (req, res) => {
  try {
    const student = await Student.findByIdAndUpdate(
      req.params.id,
      req.body,
      { new: true, runValidators: true }
    );
    
    if (!student) {
      return res.status(404).json({ success: false, message: "Student not found" });
    }
    
    res.json({ success: true, data: student });
  } catch (error) {
    res.status(400).json({ success: false, message: error.message });
  }
});
```

Tests for PUT requests

```javascript
describe("PUT /api/students/:id", () => {
  test("should update student with valid data", async () => {
    const student = await Student.create({
      name: "Before Update",
      email: "update@test.com",
      age: 20
    });
    
    const updateData = {
      name: "After Update",
      age: 25
    };
    
    const response = await request(app)
      .put(`/api/students/${student._id}`)
      .send(updateData)
      .expect(200);
    
    expect(response.body.success).toBe(true);
    expect(response.body.data.name).toBe("After Update");
    expect(response.body.data.age).toBe(25);
    expect(response.body.data.email).toBe("update@test.com"); // Unchanged
  });
  
  test("should return 404 when updating non-existent student", async () => {
    const fakeId = "507f1f77bcf86cd799439011";
    
    const response = await request(app)
      .put(`/api/students/${fakeId}`)
      .send({ name: "Updated" })
      .expect(404);
    
    expect(response.body.success).toBe(false);
    expect(response.body.message).toBe("Student not found");
  });
});
```

---

## Testing DELETE Requests

Add DELETE route to studentRoutes.js

```javascript
// DELETE student
router.delete("/:id", async (req, res) => {
  try {
    const student = await Student.findByIdAndDelete(req.params.id);
    
    if (!student) {
      return res.status(404).json({ success: false, message: "Student not found" });
    }
    
    res.json({ success: true, message: "Student deleted successfully" });
  } catch (error) {
    res.status(500).json({ success: false, message: error.message });
  }
});
```

Tests for DELETE requests

```javascript
describe("DELETE /api/students/:id", () => {
  test("should delete existing student", async () => {
    const student = await Student.create({
      name: "To Be Deleted",
      email: "delete@test.com",
      age: 20
    });
    
    const response = await request(app)
      .delete(`/api/students/${student._id}`)
      .expect(200);
    
    expect(response.body.success).toBe(true);
    expect(response.body.message).toBe("Student deleted successfully");
    
    // Verify student no longer exists
    const deletedStudent = await Student.findById(student._id);
    expect(deletedStudent).toBeNull();
  });
  
  test("should return 404 when deleting non-existent student", async () => {
    const fakeId = "507f1f77bcf86cd799439011";
    
    const response = await request(app)
      .delete(`/api/students/${fakeId}`)
      .expect(404);
    
    expect(response.body.success).toBe(false);
    expect(response.body.message).toBe("Student not found");
  });
});
```

---

## Testing Error Responses

Test all possible error scenarios

```javascript
describe("Error Handling Tests", () => {
  test("should return 400 for invalid student id format", async () => {
    const invalidId = "invalid-id";
    
    const response = await request(app)
      .get(`/api/students/${invalidId}`)
      .expect(500);
    
    expect(response.body.success).toBe(false);
  });
  
  test("should return 400 for age below minimum", async () => {
    const invalidStudent = {
      name: "Too Young",
      email: "young@test.com",
      age: 15
    };
    
    const response = await request(app)
      .post("/api/students")
      .send(invalidStudent)
      .expect(400);
    
    expect(response.body.success).toBe(false);
  });
  
  test("should return 400 for age above maximum", async () => {
    const invalidStudent = {
      name: "Too Old",
      email: "old@test.com",
      age: 70
    };
    
    const response = await request(app)
      .post("/api/students")
      .send(invalidStudent)
      .expect(400);
    
    expect(response.body.success).toBe(false);
  });
  
  test("should return 400 for invalid email format", async () => {
    const invalidStudent = {
      name: "Invalid Email",
      email: "not-an-email",
      age: 25
    };
    
    const response = await request(app)
      .post("/api/students")
      .send(invalidStudent)
      .expect(400);
    
    expect(response.body.success).toBe(false);
  });
});
```

---

## Before and After Hooks

Hooks run at specific times during tests

```javascript
describe("Student API with Hooks", () => {
  // Runs once before all tests in this describe block
  beforeAll(async () => {
    console.log("Setting up test database");
    // Connect to test database
  });
  
  // Runs before each test
  beforeEach(async () => {
    console.log("Cleaning database before test");
    await Student.deleteMany();
    
    // Create common test data
    await Student.create({
      name: "Common Student",
      email: "common@test.com",
      age: 20
    });
  });
  
  // Runs after each test
  afterEach(async () => {
    console.log("Cleaning up after test");
    await Student.deleteMany();
  });
  
  // Runs once after all tests
  afterAll(async () => {
    console.log("Closing database connection");
    await mongoose.disconnect();
  });
  
  test("test 1", async () => {
    // This test sees the common student
    const students = await Student.find();
    expect(students.length).toBe(1);
  });
  
  test("test 2", async () => {
    // This test also sees the common student
    const students = await Student.find();
    expect(students.length).toBe(1);
  });
});
```

---

## Complete Testing Example

tests/student.test.js (complete file)

```javascript
const request = require("supertest");
const mongoose = require("mongoose");
const { MongoMemoryServer } = require("mongodb-memory-server");
const app = require("../server");
const Student = require("../models/Student");

let mongoServer;

describe("Student API Complete Tests", () => {
  beforeAll(async () => {
    mongoServer = await MongoMemoryServer.create();
    const uri = mongoServer.getUri();
    await mongoose.connect(uri);
  });
  
  afterEach(async () => {
    const collections = mongoose.connection.collections;
    for (const key in collections) {
      await collections[key].deleteMany();
    }
  });
  
  afterAll(async () => {
    await mongoose.disconnect();
    await mongoServer.stop();
  });
  
  describe("GET /api/students", () => {
    test("GET all students - success", async () => {
      await Student.create([
        { name: "Student 1", email: "s1@test.com", age: 20 },
        { name: "Student 2", email: "s2@test.com", age: 22 }
      ]);
      
      const response = await request(app)
        .get("/api/students")
        .expect(200);
      
      expect(response.body.success).toBe(true);
      expect(response.body.count).toBe(2);
      expect(Array.isArray(response.body.data)).toBe(true);
    });
    
    test("GET all students - empty array", async () => {
      const response = await request(app)
        .get("/api/students")
        .expect(200);
      
      expect(response.body.success).toBe(true);
      expect(response.body.count).toBe(0);
      expect(response.body.data).toEqual([]);
    });
  });
  
  describe("GET /api/students/:id", () => {
    test("GET student by id - success", async () => {
      const student = await Student.create({
        name: "Find Me",
        email: "find@test.com",
        age: 25
      });
      
      const response = await request(app)
        .get(`/api/students/${student._id}`)
        .expect(200);
      
      expect(response.body.success).toBe(true);
      expect(response.body.data.name).toBe("Find Me");
      expect(response.body.data.email).toBe("find@test.com");
    });
    
    test("GET student by id - not found", async () => {
      const fakeId = new mongoose.Types.ObjectId();
      
      const response = await request(app)
        .get(`/api/students/${fakeId}`)
        .expect(404);
      
      expect(response.body.success).toBe(false);
      expect(response.body.message).toBe("Student not found");
    });
  });
  
  describe("POST /api/students", () => {
    test("POST create student - success", async () => {
      const newStudent = {
        name: "New Student",
        email: "new@test.com",
        age: 23,
        course: "Computer Science"
      };
      
      const response = await request(app)
        .post("/api/students")
        .send(newStudent)
        .expect(201);
      
      expect(response.body.success).toBe(true);
      expect(response.body.data.name).toBe("New Student");
      expect(response.body.data.email).toBe("new@test.com");
      expect(response.body.data).toHaveProperty("_id");
    });
    
    test("POST create student - missing required fields", async () => {
      const response = await request(app)
        .post("/api/students")
        .send({ name: "No Email" })
        .expect(400);
      
      expect(response.body.success).toBe(false);
      expect(response.body.message).toBe("Name and email are required");
    });
    
    test("POST create student - duplicate email", async () => {
      await Student.create({
        name: "First User",
        email: "duplicate@test.com",
        age: 20
      });
      
      const response = await request(app)
        .post("/api/students")
        .send({
          name: "Second User",
          email: "duplicate@test.com",
          age: 22
        })
        .expect(400);
      
      expect(response.body.success).toBe(false);
      expect(response.body.message).toBe("Email already exists");
    });
  });
  
  describe("PUT /api/students/:id", () => {
    test("PUT update student - success", async () => {
      const student = await Student.create({
        name: "Before",
        email: "update@test.com",
        age: 20
      });
      
      const response = await request(app)
        .put(`/api/students/${student._id}`)
        .send({ name: "After", age: 30 })
        .expect(200);
      
      expect(response.body.success).toBe(true);
      expect(response.body.data.name).toBe("After");
      expect(response.body.data.age).toBe(30);
      expect(response.body.data.email).toBe("update@test.com");
    });
    
    test("PUT update student - not found", async () => {
      const fakeId = new mongoose.Types.ObjectId();
      
      const response = await request(app)
        .put(`/api/students/${fakeId}`)
        .send({ name: "Updated" })
        .expect(404);
      
      expect(response.body.success).toBe(false);
      expect(response.body.message).toBe("Student not found");
    });
  });
  
  describe("DELETE /api/students/:id", () => {
    test("DELETE student - success", async () => {
      const student = await Student.create({
        name: "To Delete",
        email: "delete@test.com",
        age: 20
      });
      
      const response = await request(app)
        .delete(`/api/students/${student._id}`)
        .expect(200);
      
      expect(response.body.success).toBe(true);
      expect(response.body.message).toBe("Student deleted successfully");
      
      const deletedStudent = await Student.findById(student._id);
      expect(deletedStudent).toBeNull();
    });
    
    test("DELETE student - not found", async () => {
      const fakeId = new mongoose.Types.ObjectId();
      
      const response = await request(app)
        .delete(`/api/students/${fakeId}`)
        .expect(404);
      
      expect(response.body.success).toBe(false);
      expect(response.body.message).toBe("Student not found");
    });
  });
});
```

---

## Running Tests

Run tests once

```bash
npm test
```

Run tests in watch mode (rerun on file changes)

```bash
npm run test:watch
```

Run tests with coverage report

```bash
npm run test:coverage
```

Example coverage output

```text
------------------|---------|----------|---------|---------|
File              | % Stmts | % Branch | % Funcs | % Lines |
------------------|---------|----------|---------|---------|
All files         |   85.71 |    75.00 |   90.00 |   85.71 |
 models           |   84.21 |    80.00 |   85.71 |   84.21 |
  Student.js      |   84.21 |    80.00 |   85.71 |   84.21 |
 routes           |   87.50 |    70.00 |   100.0 |   87.50 |
  studentRoutes.js|   87.50 |    70.00 |   100.0 |   87.50 |
------------------|---------|----------|---------|---------|
```

---

## Practice Exercises

### Exercise 1

Write tests for a Products API

Test all CRUD operations

### Exercise 2

Write tests for authentication

Test register, login, protected routes

### Exercise 3

Write tests for pagination

Test page and limit parameters

### Exercise 4

Write tests for filtering

Test filter by category, price range

### Exercise 5

Achieve 90% code coverage

Write tests to cover edge cases

---

## Interview Questions

### What is API testing

API testing is verifying that API endpoints work correctly by sending requests and checking responses

### What is Jest

Jest is a JavaScript testing framework with assertions, test runner, and mocking

### What is Supertest

Supertest is a library for testing HTTP endpoints in Node.js

### What is the difference between unit tests and API tests

Unit tests test small pieces of code in isolation
API tests test entire endpoints including database and network

### What are beforeAll and afterAll hooks

They run once before and after all tests in a describe block

### What are beforeEach and afterEach hooks

They run before and after each individual test

### Why use an in-memory database for testing

So tests don't affect your real database and run faster

### What is code coverage

Code coverage measures what percentage of your code is tested

---

## Summary

In this session, you learned

* What API testing is
* Why testing is important
* Types of tests (unit, integration, API)
* What Jest and Supertest are
* How to install and set up tests
* How to test GET, POST, PUT, DELETE requests
* How to test error responses
* How to use before/after hooks
* How to set up test database
* How to run tests and check coverage

Testing ensures your API works correctly and prevents bugs

---