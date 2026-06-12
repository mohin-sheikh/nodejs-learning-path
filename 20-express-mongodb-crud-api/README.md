## Table of Contents

* [Project Overview](#project-overview)
* [Project Structure](#project-structure)
* [Setting Up the Project](#setting-up-the-project)
* [Creating the Student Model](#creating-the-student-model)
* [Creating Routes](#creating-routes)
* [Creating Controllers](#creating-controllers)
* [Putting It All Together](#putting-it-all-together)
* [Testing the API](#testing-the-api)
* [Adding More Features](#adding-more-features)
* [Practice Exercises](#practice-exercises)
* [Interview Questions](#interview-questions)
* [Summary](#summary)

---

## Project Overview

We will build a complete Student Management API

This API will have

```text
Create a new student
Get all students
Get one student by id
Update a student
Delete a student
Search students by name
Filter students by course
```

All data will be stored in MongoDB using Mongoose

This is what a real backend API looks like

---

## Project Structure

Here is how we will organize our code

```text
student-management-api/
│
├── models/
│   └── Student.js
│
├── controllers/
│   └── studentController.js
│
├── routes/
│   └── studentRoutes.js
│
├── .env
├── server.js
└── package.json
```

Why this structure

```text
models/ -> Database schemas
controllers/ -> Business logic
routes/ -> API endpoints
server.js -> Main application file
```

This is called separation of concerns

Each folder has one responsibility

---

## Setting Up the Project

Create the project

```bash
mkdir student-management-api
cd student-management-api
npm init -y
```

Install packages

```bash
npm install express mongoose dotenv
```

Create .env file

```text
PORT=5000
MONGODB_URI=mongodb+srv://yourusername:yourpassword@cluster0.abc123.mongodb.net/
DB_NAME=student_management
```

Create the folders

```bash
mkdir models controllers routes
```

Now let us create each file

---

## Creating the Student Model

Create models/Student.js

```javascript
const mongoose = require("mongoose");

const studentSchema = new mongoose.Schema({
  name: {
    type: String,
    required: [true, "Student name is required"],
    trim: true,
    minlength: [2, "Name must be at least 2 characters"],
    maxlength: [50, "Name cannot exceed 50 characters"]
  },
  age: {
    type: Number,
    required: [true, "Student age is required"],
    min: [18, "Age must be at least 18"],
    max: [60, "Age cannot exceed 60"]
  },
  course: {
    type: String,
    required: [true, "Course name is required"],
    enum: {
      values: ["Computer Science", "Mathematics", "Physics", "Chemistry", "Biology", "Engineering"],
      message: "Course must be a valid option"
    }
  },
  email: {
    type: String,
    required: [true, "Email is required"],
    unique: true,
    lowercase: true,
    trim: true,
    match: [/^\w+([\.-]?\w+)*@\w+([\.-]?\w+)*(\.\w{2,3})+$/, "Please enter a valid email"]
  },
  phoneNumber: {
    type: String,
    required: [true, "Phone number is required"],
    match: [/^\d{10}$/, "Phone number must be 10 digits"]
  },
  address: {
    street: String,
    city: String,
    zipCode: String
  },
  isActive: {
    type: Boolean,
    default: true
  }
}, {
  timestamps: true
});

module.exports = mongoose.model("Student", studentSchema);
```

---

## Creating Controllers

Controllers contain the business logic

Create controllers/studentController.js

```javascript
const Student = require("../models/Student");

// Get all students
const getAllStudents = async (req, res) => {
  try {
    const students = await Student.find();
    
    res.status(200).json({
      success: true,
      count: students.length,
      data: students
    });
  } catch (error) {
    res.status(500).json({
      success: false,
      message: "Server error",
      error: error.message
    });
  }
};

// Get single student by id
const getStudentById = async (req, res) => {
  try {
    const student = await Student.findById(req.params.id);
    
    if (!student) {
      return res.status(404).json({
        success: false,
        message: "Student not found"
      });
    }
    
    res.status(200).json({
      success: true,
      data: student
    });
  } catch (error) {
    res.status(500).json({
      success: false,
      message: "Server error",
      error: error.message
    });
  }
};

// Create new student
const createStudent = async (req, res) => {
  try {
    const student = await Student.create(req.body);
    
    res.status(201).json({
      success: true,
      data: student
    });
  } catch (error) {
    // Handle duplicate email error
    if (error.code === 11000) {
      return res.status(400).json({
        success: false,
        message: "Email already exists"
      });
    }
    
    res.status(400).json({
      success: false,
      message: error.message
    });
  }
};

// Update student
const updateStudent = async (req, res) => {
  try {
    const student = await Student.findByIdAndUpdate(
      req.params.id,
      req.body,
      {
        new: true,
        runValidators: true
      }
    );
    
    if (!student) {
      return res.status(404).json({
        success: false,
        message: "Student not found"
      });
    }
    
    res.status(200).json({
      success: true,
      data: student
    });
  } catch (error) {
    res.status(400).json({
      success: false,
      message: error.message
    });
  }
};

// Delete student
const deleteStudent = async (req, res) => {
  try {
    const student = await Student.findByIdAndDelete(req.params.id);
    
    if (!student) {
      return res.status(404).json({
        success: false,
        message: "Student not found"
      });
    }
    
    res.status(200).json({
      success: true,
      message: "Student deleted successfully"
    });
  } catch (error) {
    res.status(500).json({
      success: false,
      message: "Server error",
      error: error.message
    });
  }
};

// Search students by name
const searchStudents = async (req, res) => {
  try {
    const { name } = req.query;
    
    const students = await Student.find({
      name: { $regex: name, $options: "i" }
    });
    
    res.status(200).json({
      success: true,
      count: students.length,
      data: students
    });
  } catch (error) {
    res.status(500).json({
      success: false,
      message: "Server error",
      error: error.message
    });
  }
};

// Filter students by course
const filterByCourse = async (req, res) => {
  try {
    const { course } = req.query;
    
    const students = await Student.find({ course: course });
    
    res.status(200).json({
      success: true,
      count: students.length,
      data: students
    });
  } catch (error) {
    res.status(500).json({
      success: false,
      message: "Server error",
      error: error.message
    });
  }
};

// Get active students only
const getActiveStudents = async (req, res) => {
  try {
    const students = await Student.find({ isActive: true });
    
    res.status(200).json({
      success: true,
      count: students.length,
      data: students
    });
  } catch (error) {
    res.status(500).json({
      success: false,
      message: "Server error",
      error: error.message
    });
  }
};

module.exports = {
  getAllStudents,
  getStudentById,
  createStudent,
  updateStudent,
  deleteStudent,
  searchStudents,
  filterByCourse,
  getActiveStudents
};
```

---

## Creating Routes

Routes connect URLs to controllers

Create routes/studentRoutes.js

```javascript
const express = require("express");
const router = express.Router();

const {
  getAllStudents,
  getStudentById,
  createStudent,
  updateStudent,
  deleteStudent,
  searchStudents,
  filterByCourse,
  getActiveStudents
} = require("../controllers/studentController");

// Public routes
router.get("/", getAllStudents);
router.get("/active", getActiveStudents);
router.get("/search", searchStudents);
router.get("/filter", filterByCourse);
router.get("/:id", getStudentById);

// Write operations
router.post("/", createStudent);
router.put("/:id", updateStudent);
router.delete("/:id", deleteStudent);

module.exports = router;
```

Notice the order of routes

```text
/active comes before /:id
/search comes before /:id
/filter comes before /:id
```

Because if /:id comes first, it will treat "active" as an id

---

## Putting It All Together

Create server.js

```javascript
require("dotenv").config();
const express = require("express");
const mongoose = require("mongoose");

const studentRoutes = require("./routes/studentRoutes");

const app = express();

// Middleware
app.use(express.json());
app.use(express.urlencoded({ extended: true }));

// Routes
app.use("/api/students", studentRoutes);

// Home route
app.get("/", (req, res) => {
  res.json({
    message: "Welcome to Student Management API",
    endpoints: {
      getAllStudents: "GET /api/students",
      getStudentById: "GET /api/students/:id",
      createStudent: "POST /api/students",
      updateStudent: "PUT /api/students/:id",
      deleteStudent: "DELETE /api/students/:id",
      searchStudents: "GET /api/students/search?name=john",
      filterByCourse: "GET /api/students/filter?course=Computer Science",
      getActiveStudents: "GET /api/students/active"
    }
  });
});

// 404 handler for unknown routes
app.use("*", (req, res) => {
  res.status(404).json({
    success: false,
    message: `Route ${req.originalUrl} not found`
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

// Connect to MongoDB and start server
const PORT = process.env.PORT || 5000;
const MONGODB_URI = `${process.env.MONGODB_URI}/${process.env.DB_NAME}`;

mongoose.connect(MONGODB_URI)
  .then(() => {
    console.log("Connected to MongoDB");
    app.listen(PORT, () => {
      console.log(`Server running on port ${PORT}`);
      console.log(`http://localhost:${PORT}`);
    });
  })
  .catch((error) => {
    console.error("MongoDB connection failed", error);
    process.exit(1);
  });
```

---

## Testing the API

Start the server

```bash
node server.js
```

You will see

```text
Connected to MongoDB
Server running on port 5000
http://localhost:5000
```

Test with curl commands

Create a student (POST)

```bash
curl -X POST http://localhost:5000/api/students \
  -H "Content-Type: application/json" \
  -d '{
    "name": "John Doe",
    "age": 20,
    "course": "Computer Science",
    "email": "john@example.com",
    "phoneNumber": "1234567890",
    "address": {
      "street": "123 Main St",
      "city": "New York",
      "zipCode": "10001"
    }
  }'
```

Get all students (GET)

```bash
curl http://localhost:5000/api/students
```

Get one student (GET)

```bash
curl http://localhost:5000/api/students/STUDENT_ID_HERE
```

Search students by name (GET)

```bash
curl "http://localhost:5000/api/students/search?name=john"
```

Filter by course (GET)

```bash
curl "http://localhost:5000/api/students/filter?course=Computer%20Science"
```

Get active students (GET)

```bash
curl http://localhost:5000/api/students/active
```

Update a student (PUT)

```bash
curl -X PUT http://localhost:5000/api/students/STUDENT_ID_HERE \
  -H "Content-Type: application/json" \
  -d '{
    "name": "John Updated",
    "age": 21
  }'
```

Delete a student (DELETE)

```bash
curl -X DELETE http://localhost:5000/api/students/STUDENT_ID_HERE
```

---

## Adding More Features

Let us add pagination to getAllStudents

Update the getAllStudents controller

```javascript
const getAllStudents = async (req, res) => {
  try {
    const page = parseInt(req.query.page) || 1;
    const limit = parseInt(req.query.limit) || 10;
    const skip = (page - 1) * limit;
    
    const students = await Student.find()
      .skip(skip)
      .limit(limit);
    
    const total = await Student.countDocuments();
    
    res.status(200).json({
      success: true,
      count: students.length,
      total: total,
      page: page,
      pages: Math.ceil(total / limit),
      data: students
    });
  } catch (error) {
    res.status(500).json({
      success: false,
      message: "Server error",
      error: error.message
    });
  }
};
```

Now you can use

```text
GET /api/students?page=1&limit=5
GET /api/students?page=2&limit=5
```

---

## Practice Exercises

### Exercise 1

Add a new field to the Student model

```text
grade: A, B, C, D, F
```

Add validation for grade

### Exercise 2

Create a new route

```text
GET /api/students/grade/:grade
```

Return all students with that grade

### Exercise 3

Add sorting feature to getAllStudents

```text
GET /api/students?sort=name
GET /api/students?sort=-age (descending)
```

### Exercise 4

Create a new model called Course

```text
name
description
duration
price
```

Create CRUD routes for Course

### Exercise 5

Add a relationship between Student and Course

Each student can enroll in multiple courses

---

## Interview Questions

### Why do we separate routes, controllers, and models

To keep code organized, maintainable, and follow separation of concerns

### What is the purpose of controllers

Controllers contain the business logic for handling requests

### What is the purpose of routes

Routes define the API endpoints and connect them to controllers

### Why is route order important

Express matches routes in order, so specific routes should come before parameter routes

### What is pagination and why do we use it

Pagination splits large result sets into smaller pages to improve performance

### What does $regex do in MongoDB

$regex allows pattern matching for string searches

### What does the option $options: "i" do

It makes the search case insensitive

---

## Summary

In this session, you learned

* How to structure a real backend project
* How to separate models, controllers, and routes
* How to build a complete CRUD API with Express and MongoDB
* How to add search and filter features
* How to add pagination
* How to test all API endpoints
* How to handle errors properly

You have built a production-ready backend API

---