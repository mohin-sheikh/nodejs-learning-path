## Table of Contents

* [What is MVC](#what-is-mvc)
* [Why Use MVC](#why-use-mvc)
* [MVC Components Explained](#mvc-components-explained)
* [Traditional vs MVC Structure](#traditional-vs-mvc-structure)
* [Setting Up MVC Project](#setting-up-mvc-project)
* [Model Layer](#model-layer)
* [View Layer](#view-layer)
* [Controller Layer](#controller-layer)
* [Complete MVC Example](#complete-mvc-example)
* [Practice Exercises](#practice-exercises)
* [Interview Questions](#interview-questions)
* [Summary](#summary)

---

## What is MVC

MVC is a design pattern

It stands for Model View Controller

It helps organize your code into three separate parts

Think of it like a restaurant

```text
Model -> Kitchen (where food is prepared)
View -> Dining table (where food is presented)
Controller -> Waiter (takes order and brings food)
```

When a customer comes to a restaurant

```text
Customer tells order to Waiter (Controller)
Waiter takes order to Kitchen (Model)
Kitchen prepares food (Model)
Waiter brings food to Dining table (View)
Customer sees the food (View)
```

Same in web applications

```text
Browser sends request to Controller
Controller asks Model for data
Model gets data from database
Controller sends data to View
View shows data to user
```

---

## Why Use MVC

Problems without MVC

```text
All code in one file
Hard to find where things are
Difficult to make changes
Cannot reuse code
Hard to work in teams
```

Benefits of MVC

```text
Separation of concerns
Code is organized and clean
Easy to find and fix bugs
Can reuse code in different places
Multiple developers can work together
Easy to scale the application
```

Example of bad code without MVC

```javascript
// Everything mixed together
app.get("/students", async (req, res) => {
  // Database logic here
  const students = await Student.find();
  
  // Business logic here
  const activeStudents = students.filter(s => s.isActive);
  
  // HTML generation here
  let html = "<html><body>";
  activeStudents.forEach(s => {
    html += `<p>${s.name}</p>`;
  });
  html += "</body></html>";
  
  res.send(html);
});
```

With MVC, each part has its own responsibility

---

## MVC Components Explained

### Model

The Model handles data and database operations

```text
What it does -> Talks to database
What it contains -> Schema, validation, queries
What it does NOT do -> Handle requests, show HTML
```

Example of Model work

```text
Find all students
Save a new student
Update student data
Delete a student
```

### View

The View handles what the user sees

```text
What it does -> Shows data to user
What it contains -> HTML, JSON, EJS templates
What it does NOT do -> Talk to database, handle logic
```

Example of View work

```text
Show student list in HTML
Send JSON response
Render a form
Display error messages
```

### Controller

The Controller handles the logic

```text
What it does -> Receives requests, processes data
What it contains -> Validation, business rules, calls Model and View
What it does NOT do -> Direct database queries, HTML generation
```

Example of Controller work

```text
Get request from /students
Ask Model for students
Send students to View
Send response back to browser
```

---

## Traditional vs MVC Structure

Traditional structure (all in one file)

```text
server.js
    |
    -> Routes
    -> Database queries
    -> Business logic
    -> HTML generation
    -> Error handling
```

Problems

```text
500 lines in one file
Hard to read
Hard to debug
Cannot reuse code
```

MVC structure

```text
project/
    |
    ├── models/
    |      -> Student.js
    |      -> Course.js
    |
    ├── views/
    |      -> studentListView.ejs
    |      -> studentDetailView.ejs
    |
    ├── controllers/
    |      -> studentController.js
    |      -> courseController.js
    |
    ├── routes/
    |      -> studentRoutes.js
    |
    └── server.js
```

Benefits

```text
Each file has less than 100 lines
Easy to find what you need
Easy to debug
Code is reusable
```

---

## Setting Up MVC Project

Create a new project

```bash
mkdir mvc-api
cd mvc-api
npm init -y
```

Install packages

```bash
npm install express mongoose dotenv
```

Create folder structure

```bash
mkdir models controllers routes
```

Create .env file

```text
PORT=5000
MONGODB_URI=mongodb+srv://yourusername:yourpassword@cluster0.abc123.mongodb.net/
DB_NAME=mvc_school
```

Now let us build each layer

---

## Model Layer

The Model layer is responsible for data

Create models/Student.js

```javascript
const mongoose = require("mongoose");

const studentSchema = new mongoose.Schema({
  name: {
    type: String,
    required: true,
    trim: true
  },
  age: {
    type: Number,
    required: true,
    min: 18,
    max: 60
  },
  course: {
    type: String,
    required: true
  },
  email: {
    type: String,
    required: true,
    unique: true,
    lowercase: true
  },
  grade: {
    type: String,
    enum: ["A", "B", "C", "D", "F"],
    default: "B"
  },
  isActive: {
    type: Boolean,
    default: true
  }
}, {
  timestamps: true
});

// Custom method in Model
studentSchema.methods.getSummary = function() {
  return `${this.name} is studying ${this.course} and got grade ${this.grade}`;
};

// Static method in Model
studentSchema.statics.findByCourse = function(courseName) {
  return this.find({ course: courseName });
};

module.exports = mongoose.model("Student", studentSchema);
```

Notice the Model has

```text
Schema definition -> Structure of data
Validation rules -> Required, min, max, enum
Custom methods -> getSummary()
Static methods -> findByCourse()
```

---

## View Layer

The View layer is what the user sees

For APIs, the View is usually JSON

But we can also create a simple View using EJS

First install EJS

```bash
npm install ejs
```

Create a views folder

```bash
mkdir views
```

Create views/studentListView.ejs

```html
<!DOCTYPE html>
<html>
<head>
    <title>Student List</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            margin: 40px;
        }
        .student-card {
            border: 1px solid #ddd;
            padding: 15px;
            margin: 10px 0;
            border-radius: 5px;
        }
        .student-name {
            color: blue;
            font-size: 20px;
        }
        .student-info {
            color: #666;
            margin-top: 10px;
        }
    </style>
</head>
<body>
    <h1>Student List</h1>
    <p>Total Students: <%= students.length %></p>
    
    <% students.forEach(function(student) { %>
        <div class="student-card">
            <div class="student-name"><%= student.name %></div>
            <div class="student-info">
                Age: <%= student.age %> |
                Course: <%= student.course %> |
                Grade: <%= student.grade %>
            </div>
            <div class="student-info">
                Email: <%= student.email %>
            </div>
        </div>
    <% }); %>
    
    <% if(students.length === 0) { %>
        <p>No students found</p>
    <% } %>
</body>
</html>
```

Create views/studentDetailView.ejs

```html
<!DOCTYPE html>
<html>
<head>
    <title>Student Details</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            margin: 40px;
        }
        .details-card {
            border: 1px solid #ddd;
            padding: 20px;
            border-radius: 5px;
            max-width: 500px;
        }
        .field {
            margin: 10px 0;
            padding: 10px;
            background: #f5f5f5;
        }
        .field-label {
            font-weight: bold;
        }
    </style>
</head>
<body>
    <h1>Student Details</h1>
    
    <div class="details-card">
        <div class="field">
            <div class="field-label">Name</div>
            <div><%= student.name %></div>
        </div>
        
        <div class="field">
            <div class="field-label">Age</div>
            <div><%= student.age %></div>
        </div>
        
        <div class="field">
            <div class="field-label">Course</div>
            <div><%= student.course %></div>
        </div>
        
        <div class="field">
            <div class="field-label">Email</div>
            <div><%= student.email %></div>
        </div>
        
        <div class="field">
            <div class="field-label">Grade</div>
            <div><%= student.grade %></div>
        </div>
        
        <div class="field">
            <div class="field-label">Status</div>
            <div><%= student.isActive ? "Active" : "Inactive" %></div>
        </div>
    </div>
    
    <a href="/students">Back to Student List</a>
</body>
</html>
```

The View layer

```text
Shows data in HTML format
Uses EJS templating
No database logic here
Only presentation logic
```

---

## Controller Layer

The Controller handles requests and coordinates Model and View

Create controllers/studentController.js

```javascript
const Student = require("../models/Student");

// Get all students and render HTML view
const renderAllStudents = async (req, res) => {
  try {
    const students = await Student.find();
    
    res.render("studentListView", {
      students: students,
      title: "Student Management System"
    });
  } catch (error) {
    res.status(500).send("Server Error");
  }
};

// Get single student and render HTML view
const renderStudentById = async (req, res) => {
  try {
    const student = await Student.findById(req.params.id);
    
    if (!student) {
      return res.status(404).send("Student not found");
    }
    
    res.render("studentDetailView", {
      student: student
    });
  } catch (error) {
    res.status(500).send("Server Error");
  }
};

// API version - Get all students as JSON
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
      message: error.message
    });
  }
};

// API version - Get single student as JSON
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
      message: error.message
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
      { new: true, runValidators: true }
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
      message: error.message
    });
  }
};

// Get students by course using static method
const getStudentsByCourse = async (req, res) => {
  try {
    const courseName = req.params.course;
    const students = await Student.findByCourse(courseName);
    
    res.status(200).json({
      success: true,
      count: students.length,
      data: students
    });
  } catch (error) {
    res.status(500).json({
      success: false,
      message: error.message
    });
  }
};

module.exports = {
  renderAllStudents,
  renderStudentById,
  getAllStudents,
  getStudentById,
  createStudent,
  updateStudent,
  deleteStudent,
  getStudentsByCourse
};
```

Notice the Controller

```text
Has both HTML rendering and JSON methods
Uses try-catch for error handling
Calls Model methods
Sends response to View
No direct database queries here
```

---

## Complete MVC Example

Create routes/studentRoutes.js

```javascript
const express = require("express");
const router = express.Router();

const {
  renderAllStudents,
  renderStudentById,
  getAllStudents,
  getStudentById,
  createStudent,
  updateStudent,
  deleteStudent,
  getStudentsByCourse
} = require("../controllers/studentController");

// HTML View routes
router.get("/view", renderAllStudents);
router.get("/view/:id", renderStudentById);

// API routes
router.get("/", getAllStudents);
router.get("/course/:course", getStudentsByCourse);
router.get("/:id", getStudentById);
router.post("/", createStudent);
router.put("/:id", updateStudent);
router.delete("/:id", deleteStudent);

module.exports = router;
```

Create server.js

```javascript
require("dotenv").config();
const express = require("express");
const mongoose = require("mongoose");
const path = require("path");

const studentRoutes = require("./routes/studentRoutes");

const app = express();

// Set up view engine
app.set("view engine", "ejs");
app.set("views", path.join(__dirname, "views"));

// Middleware
app.use(express.json());
app.use(express.urlencoded({ extended: true }));

// Routes
app.use("/students", studentRoutes);

// Home route
app.get("/", (req, res) => {
  res.json({
    message: "MVC API is running",
    endpoints: {
      htmlViews: {
        allStudents: "GET /students/view",
        studentDetails: "GET /students/view/:id"
      },
      apiEndpoints: {
        getAllStudents: "GET /students",
        getStudentById: "GET /students/:id",
        getByCourse: "GET /students/course/:course",
        createStudent: "POST /students",
        updateStudent: "PUT /students/:id",
        deleteStudent: "DELETE /students/:id"
      }
    }
  });
});

// Connect to MongoDB
const MONGODB_URI = `${process.env.MONGODB_URI}/${process.env.DB_NAME}`;

mongoose.connect(MONGODB_URI)
  .then(() => {
    console.log("Connected to MongoDB");
    
    const PORT = process.env.PORT || 5000;
    app.listen(PORT, () => {
      console.log(`Server running on port ${PORT}`);
      console.log(`HTML Views: http://localhost:${PORT}/students/view`);
      console.log(`API: http://localhost:${PORT}/students`);
    });
  })
  .catch(err => {
    console.error("MongoDB connection failed", err);
    process.exit(1);
  });
```

---

## How MVC Works Together

Flow of a request

```text
1. Browser sends GET request to /students/view
                |
                v
2. Routes file receives the request
                |
                v
3. Routes calls renderAllStudents controller
                |
                v
4. Controller calls Student.find() (Model)
                |
                v
5. Model gets data from MongoDB
                |
                v
6. Controller receives data
                |
                v
7. Controller sends data to View (EJS template)
                |
                v
8. View generates HTML
                |
                v
9. Response sent back to browser
```

Each layer has one job

```text
Model -> Get data
View -> Show data
Controller -> Coordinate everything
```

---

## Practice Exercises

### Exercise 1

Create a Course model

```text
name
description
duration
price
```

### Exercise 2

Create a controller for Course with

```text
getAllCourses
getCourseById
createCourse
updateCourse
deleteCourse
```

### Exercise 3

Create a view for Course list using EJS

### Exercise 4

Add a relationship between Student and Course

Each student can enroll in multiple courses

### Exercise 5

Create a dashboard view that shows

```text
Total students
Total courses
Active students
Average student age
```

---

## Interview Questions

### What does MVC stand for

Model View Controller

### What is the role of Model in MVC

Model handles data and database operations

### What is the role of View in MVC

View handles what the user sees (HTML, JSON)

### What is the role of Controller in MVC

Controller handles requests, processes data, and coordinates Model and View

### Why do we use MVC pattern

To separate concerns, organize code, and make applications easier to maintain

### Can MVC be used for both APIs and web applications

Yes
MVC works for both JSON APIs and HTML web applications

### What is the difference between traditional coding and MVC

Traditional coding mixes everything in one file
MVC separates code into three distinct layers

---

## Summary

In this session, you learned

* What MVC architecture is
* Why we use MVC pattern
* The three components: Model, View, Controller
* How each component has its own responsibility
* How Model handles data
* How View handles presentation
* How Controller coordinates everything
* How to build an MVC application
* How to create both HTML views and JSON APIs

MVC is the standard pattern for professional backend applications

---