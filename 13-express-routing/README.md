## Table of Contents

* [What is Routing](#what-is-routing)
* [Basic Routes in Express](#basic-routes-in-express)
* [Route Parameters](#route-parameters)
* [Query Parameters](#query-query-parameters)
* [Multiple Route Parameters](#multiple-route-parameters)
* [Optional Route Parameters](#optional-route-parameters)
* [Route with Multiple HTTP Methods](#route-with-multiple-http-methods)
* [Organizing Routes](#organizing-routes)
* [Route Files](#route-files)
* [Complete Example](#complete-example)
* [Practice Exercises](#practice-exercises)
* [Interview Questions](#interview-questions)
* [Summary](#summary)

---

## What is Routing

Routing means deciding what happens when someone visits a URL

Example

```text
/students -> Show all students
/students/1 -> Show student with id 1
/about -> Show about page
```

Routing connects a URL to a function

Think of it like a receptionist

```text
Visitor comes to /students
Receptionist sends them to the students function

Visitor comes to /about
Receptionist sends them to the about function
```

Express makes routing very easy

---

## Basic Routes in Express

Here are the most common routes

```javascript
const express = require("express");
const app = express();

// Home page
app.get("/", (req, res) => {
  res.send("Welcome to Home Page");
});

// About page
app.get("/about", (req, res) => {
  res.send("About Us");
});

// Contact page
app.get("/contact", (req, res) => {
  res.send("Contact Us");
});

// Products page
app.get("/products", (req, res) => {
  res.send("List of Products");
});

app.listen(3000);
```

Each route has three parts

```text
app.get() -> HTTP method
"/about" -> URL path
(req, res) => {} -> Function that runs
```

You can use any URL you want

---

## Route Parameters

Route parameters let you capture values from the URL

Use a colon : before the parameter name

Example

```javascript
app.get("/students/:id", (req, res) => {
  const id = req.params.id;
  res.send(`Student ID is ${id}`);
});
```

If someone visits /students/5

```text
req.params.id will be 5
```

If someone visits /students/100

```text
req.params.id will be 100
```

You can name the parameter anything

```javascript
app.get("/products/:productId", (req, res) => {
  const productId = req.params.productId;
  res.send(`Product ID is ${productId}`);
});

app.get("/users/:userId", (req, res) => {
  const userId = req.params.userId;
  res.send(`User ID is ${userId}`);
});
```

Real world example with student data

```javascript
app.get("/students/:id", (req, res) => {
  const id = parseInt(req.params.id);
  
  const student = students.find(s => s.id === id);
  
  if (student) {
    res.json(student);
  } else {
    res.status(404).json({ message: "Student not found" });
  }
});
```

---

## Query Parameters

Query parameters come after a question mark in the URL

Example URL

```text
/students?course=Computer&age=20
```

The query parameters are

```text
course = Computer
age = 20
```

Express puts all query parameters in req.query

```javascript
app.get("/students", (req, res) => {
  const course = req.query.course;
  const age = req.query.age;
  
  res.json({ course, age });
});
```

Visit /students?course=Computer&age=20

You will get

```json
{
  "course": "Computer",
  "age": "20"
}
```

Real world example with filtering

```javascript
app.get("/students", (req, res) => {
  let result = students;
  
  // Filter by course if provided
  if (req.query.course) {
    result = result.filter(s => s.course === req.query.course);
  }
  
  // Filter by minimum age if provided
  if (req.query.minAge) {
    result = result.filter(s => s.age >= parseInt(req.query.minAge));
  }
  
  res.json(result);
});
```

Now you can use

```text
/students?course=Computer Science
/students?minAge=21
/students?course=Mathematics&minAge=20
```

---

## Multiple Route Parameters

You can use multiple parameters in one route

```javascript
app.get("/students/:studentId/courses/:courseId", (req, res) => {
  const studentId = req.params.studentId;
  const courseId = req.params.courseId;
  
  res.json({
    studentId: studentId,
    courseId: courseId
  });
});
```

Visit /students/5/courses/10

You will get

```json
{
  "studentId": "5",
  "courseId": "10"
}
```

This is useful for nested resources

---

## Optional Route Parameters

Sometimes you want a parameter to be optional

Add a question mark

```javascript
app.get("/products/:category/:productId?", (req, res) => {
  const category = req.params.category;
  const productId = req.params.productId;
  
  if (productId) {
    res.send(`Category ${category}, Product ${productId}`);
  } else {
    res.send(`Category ${category}, All Products`);
  }
});
```

Now both URLs work

```text
/products/electronics
/products/electronics/123
```

---

## Route with Multiple HTTP Methods

Same URL can behave differently based on HTTP method

```javascript
// GET all students
app.get("/students", (req, res) => {
  res.json(students);
});

// POST add new student
app.post("/students", (req, res) => {
  const newStudent = req.body;
  newStudent.id = students.length + 1;
  students.push(newStudent);
  res.status(201).json(newStudent);
});

// DELETE all students
app.delete("/students", (req, res) => {
  students = [];
  res.json({ message: "All students deleted" });
});
```

The URL is same /students

But behavior changes based on GET, POST, DELETE

---

## Organizing Routes

As your app grows, you will have many routes

Instead of putting all routes in one file, organize them

Project structure

```text
my-app/
├── server.js
├── routes/
│   ├── students.js
│   ├── products.js
│   └── users.js
```

Create a route file routes/students.js

```javascript
const express = require("express");
const router = express.Router();

// All student routes go here
router.get("/", (req, res) => {
  res.json({ message: "Get all students" });
});

router.get("/:id", (req, res) => {
  res.json({ message: `Get student ${req.params.id}` });
});

router.post("/", (req, res) => {
  res.json({ message: "Add new student" });
});

module.exports = router;
```

In your main server.js

```javascript
const express = require("express");
const app = express();

const studentRoutes = require("./routes/students");

app.use("/students", studentRoutes);

app.listen(3000);
```

Now when someone visits /students

Express sends it to the student routes file

This keeps your code clean

---

## Route Files Complete Example

routes/students.js

```javascript
const express = require("express");
const router = express.Router();

let students = [
  {
    id: 1,
    name: "John Doe",
    age: 20,
    course: "Computer Science"
  },
  {
    id: 2,
    name: "Jane Smith",
    age: 22,
    course: "Mathematics"
  }
];

// GET all students
router.get("/", (req, res) => {
  res.json(students);
});

// GET one student
router.get("/:id", (req, res) => {
  const id = parseInt(req.params.id);
  const student = students.find(s => s.id === id);
  
  if (student) {
    res.json(student);
  } else {
    res.status(404).json({ message: "Student not found" });
  }
});

// POST add student
router.post("/", (req, res) => {
  const newStudent = req.body;
  newStudent.id = students.length + 1;
  students.push(newStudent);
  res.status(201).json(newStudent);
});

// PUT update student
router.put("/:id", (req, res) => {
  const id = parseInt(req.params.id);
  const updatedData = req.body;
  const index = students.findIndex(s => s.id === id);
  
  if (index !== -1) {
    students[index] = { ...students[index], ...updatedData };
    res.json(students[index]);
  } else {
    res.status(404).json({ message: "Student not found" });
  }
});

// DELETE student
router.delete("/:id", (req, res) => {
  const id = parseInt(req.params.id);
  const index = students.findIndex(s => s.id === id);
  
  if (index !== -1) {
    students.splice(index, 1);
    res.json({ message: "Student deleted successfully" });
  } else {
    res.status(404).json({ message: "Student not found" });
  }
});

module.exports = router;
```

server.js

```javascript
const express = require("express");
const app = express();

app.use(express.json());

const studentRoutes = require("./routes/students");

app.use("/students", studentRoutes);

app.listen(3000, () => {
  console.log("Server running on port 3000");
});
```

Now all student related code is in one file

---

## Complete Example with Multiple Route Files

Project structure

```text
my-api/
├── server.js
├── routes/
│   ├── students.js
│   ├── products.js
│   └── courses.js
```

server.js

```javascript
const express = require("express");
const app = express();

app.use(express.json());

const studentRoutes = require("./routes/students");
const productRoutes = require("./routes/products");
const courseRoutes = require("./routes/courses");

app.use("/students", studentRoutes);
app.use("/products", productRoutes);
app.use("/courses", courseRoutes);

app.listen(3000, () => {
  console.log("Server running on port 3000");
});
```

routes/products.js

```javascript
const express = require("express");
const router = express.Router();

let products = [
  {
    id: 1,
    name: "Laptop",
    price: 50000
  },
  {
    id: 2,
    name: "Mobile",
    price: 15000
  }
];

router.get("/", (req, res) => {
  res.json(products);
});

router.post("/", (req, res) => {
  const newProduct = req.body;
  newProduct.id = products.length + 1;
  products.push(newProduct);
  res.status(201).json(newProduct);
});

module.exports = router;
```

Now your code is organized and easy to maintain

---

## Route Order Matters

Express checks routes in order

Always put specific routes before general routes

Correct order

```javascript
// Specific route first
app.get("/students/new", (req, res) => {
  res.send("New student form");
});

// General route with parameter next
app.get("/students/:id", (req, res) => {
  res.send(`Student ${req.params.id}`);
});
```

Wrong order

```javascript
// This will catch /students/new
app.get("/students/:id", (req, res) => {
  res.send(`Student ${req.params.id}`);
});

// This will never run because above route catches it
app.get("/students/new", (req, res) => {
  res.send("New student form");
});
```

Remember to put specific routes first

---

## Practice Exercises

### Exercise 1

Create a route

```text
GET /users/:userId
```

Return the userId as JSON

### Exercise 2

Create a route with query parameters

```text
GET /search?q=nodejs
```

Return the search term as JSON

### Exercise 3

Create two route files

```text
routes/books.js
routes/authors.js
```

Import them in server.js

### Exercise 4

Create a route with multiple parameters

```text
GET /store/:category/:itemId
```

Return both parameters as JSON

### Exercise 5

Create a filter route

```text
GET /products?minPrice=100&maxPrice=500
```

Return only products within the price range

---

## Interview Questions

### What is routing in Express

Routing is deciding what happens when a user visits a specific URL

### What is the difference between route parameters and query parameters

Route parameters are part of the URL path like /users/5
Query parameters come after ? like /users?id=5

### How do you access route parameters

Using req.params

### How do you access query parameters

Using req.query

### Why do we organize routes into separate files

To keep code clean and maintainable

### What is the purpose of express.Router()

It helps create separate route files that can be imported into the main app

---

## Summary

In this session, you learned

* What routing is
* How to create basic routes
* How to use route parameters with :
* How to use query parameters with ?
* How to use multiple parameters
* How to make parameters optional
* How to organize routes into separate files
* How to use express.Router()
* The importance of route order

You can now build organized and clean Express applications

---