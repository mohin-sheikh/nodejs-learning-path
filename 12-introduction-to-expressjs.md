## Table of Contents

* [What is Express.js](#what-is-expressjs)
* [Why Use Express](#why-use-express)
* [Installing Express](#installing-express)
* [Your First Express Server](#your-first-express-server)
* [Express Routes](#express-routes)
* [Sending JSON Responses](#sending-json-responses)
* [GET Requests in Express](#get-requests-in-express)
* [POST Requests in Express](#post-requests-in-express)
* [PUT Requests in Express](#put-requests-in-express)
* [DELETE Requests in Express](#delete-requests-in-express)
* [URL Parameters in Express](#url-parameters-in-express)
* [Complete Student API with Express](#complete-student-api-with-express)
* [Comparing HTTP Module vs Express](#comparing-http-module-vs-express)
* [Practice Exercises](#practice-exercises)
* [Interview Questions](#interview-questions)
* [Summary](#summary)

---

## What is Express.js

Express.js is a framework for Node.js

A framework gives you ready-made tools

Think of it like this

```text
Node.js is like raw bricks and cement
Express is like a house blueprint with tools
```

Express makes building web applications easier

It is the most popular Node.js framework

Millions of developers use Express

---

## Why Use Express

Remember our server code from Session 11

We had to write many lines of code

We had to manually check req.url and req.method

We had to manually collect POST data

Express solves these problems

Benefits of Express

```text
Less code to write
Easier to understand
Built-in tools for routes
Built-in tools for POST data
Better error handling
Used by companies worldwide
```

Let me show you the difference

With HTTP module, we wrote

```javascript
if (req.url === "/students" && req.method === "GET") {
  // lots of code
}
```

With Express, we write

```javascript
app.get("/students", (req, res) => {
  // shorter code
});
```

Much cleaner

---

## Installing Express

First, create a new project

```bash
mkdir express-student-api
cd express-student-api
npm init -y
```

Now install Express

```bash
npm install express
```

This adds express to your package.json dependencies

Now create a file named server.js

---

## Your First Express Server

Here is the simplest Express server

```javascript
const express = require("express");
const app = express();

app.get("/", (req, res) => {
  res.send("Hello from Express");
});

app.listen(3000, () => {
  console.log("Server running on port 3000");
});
```

Let me explain each line

```text
const express = require("express") -> Import Express
const app = express() -> Create an Express application
app.get("/", (req, res) -> Handle GET request to /
res.send("Hello from Express") -> Send response
app.listen(3000) -> Start server on port 3000
```

Run the server

```bash
node server.js
```

Open browser and visit http://localhost:3000

You will see "Hello from Express"

---

## Express Routes

Routes in Express are very simple

```javascript
app.get("/about", (req, res) => {
  res.send("About Page");
});

app.get("/contact", (req, res) => {
  res.send("Contact Page");
});

app.post("/data", (req, res) => {
  res.send("Data received");
});
```

The pattern is always

```text
app.METHOD(PATH, HANDLER)
```

Where

```text
METHOD -> get, post, put, delete
PATH -> URL like /students, /about
HANDLER -> Function that runs when route is called
```

---

## Sending JSON Responses

In Session 11, we used JSON.stringify()

In Express, we use res.json()

```javascript
app.get("/user", (req, res) => {
  const user = {
    name: "John",
    age: 25
  };
  
  res.json(user);
});
```

Express automatically

```text
Converts object to JSON
Sets Content-Type header
Sends the response
```

Much easier than before

---

## GET Requests in Express

Remember our GET all students from Session 11

Let me show you the Express version

```javascript
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

app.get("/students", (req, res) => {
  res.json(students);
});
```

That is it

No status code to set manually (Express uses 200 by default)

No JSON.stringify()

No Content-Type header

Just res.json()

---

## POST Requests in Express

In Session 11, we had to collect data chunk by chunk

Express makes this easy with a middleware called express.json()

First, add this line before your routes

```javascript
app.use(express.json());
```

This tells Express to automatically parse JSON from request body

Now POST request looks like this

```javascript
app.post("/students", (req, res) => {
  const newStudent = req.body;
  
  newStudent.id = students.length + 1;
  
  students.push(newStudent);
  
  res.status(201).json(newStudent);
});
```

Compare this with Session 11

We no longer need

```text
let body = ""
req.on("data")
req.on("end")
JSON.parse()
```

Express does all of that for us

req.body contains the incoming data

---

## PUT Requests in Express

PUT request is also simpler

```javascript
app.put("/students/:id", (req, res) => {
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
```

Notice the :id in the URL

```text
/students/:id
```

Express captures this for us

We access it using req.params.id

No need to split the URL manually

---

## DELETE Requests in Express

DELETE request is also very clean

```javascript
app.delete("/students/:id", (req, res) => {
  const id = parseInt(req.params.id);
  
  const index = students.findIndex(s => s.id === id);
  
  if (index !== -1) {
    students.splice(index, 1);
    res.json({ message: "Student deleted successfully" });
  } else {
    res.status(404).json({ message: "Student not found" });
  }
});
```

---

## URL Parameters in Express

Express makes it easy to get data from URLs

There are two common ways

Query Parameters

```text
/students?name=John&age=20
```

Access using req.query

```javascript
app.get("/students", (req, res) => {
  const name = req.query.name;
  const age = req.query.age;
  res.json({ name, age });
});
```

Route Parameters

```text
/students/1
/students/2
```

Access using req.params

```javascript
app.get("/students/:id", (req, res) => {
  const id = req.params.id;
  res.json({ id });
});
```

---

## Complete Student API with Express

Here is the complete server.js using Express

```javascript
const express = require("express");
const app = express();

app.use(express.json());

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
  },
  {
    id: 3,
    name: "Mike Johnson",
    age: 21,
    course: "Physics"
  }
];

// GET all students
app.get("/students", (req, res) => {
  res.json(students);
});

// GET one student
app.get("/students/:id", (req, res) => {
  const id = parseInt(req.params.id);
  const student = students.find(s => s.id === id);
  
  if (student) {
    res.json(student);
  } else {
    res.status(404).json({ message: "Student not found" });
  }
});

// POST add new student
app.post("/students", (req, res) => {
  const newStudent = req.body;
  newStudent.id = students.length + 1;
  students.push(newStudent);
  res.status(201).json(newStudent);
});

// PUT update student
app.put("/students/:id", (req, res) => {
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
app.delete("/students/:id", (req, res) => {
  const id = parseInt(req.params.id);
  
  const index = students.findIndex(s => s.id === id);
  
  if (index !== -1) {
    students.splice(index, 1);
    res.json({ message: "Student deleted successfully" });
  } else {
    res.status(404).json({ message: "Student not found" });
  }
});

app.listen(3000, () => {
  console.log("Server running on port 3000");
  console.log("Student API with Express");
});
```

---

## Comparing HTTP Module vs Express

Let me show you the difference side by side

Session 11 with HTTP module

```javascript
if (req.url === "/students" && req.method === "GET") {
  res.statusCode = 200;
  res.setHeader("Content-Type", "application/json");
  res.end(JSON.stringify(students));
}
```

Express version

```javascript
app.get("/students", (req, res) => {
  res.json(students);
});
```

POST request with HTTP module required 15 lines of code

POST request with Express requires 6 lines of code

Express saves time and reduces mistakes

---

## Testing Your Express API

Run the server

```bash
node server.js
```

Test GET requests in browser

```text
http://localhost:3000/students
http://localhost:3000/students/1
```

Test POST, PUT, DELETE using Postman or curl

POST request

```bash
curl -X POST http://localhost:3000/students \
  -H "Content-Type: application/json" \
  -d '{"name":"Sarah","age":23,"course":"Biology"}'
```

PUT request

```bash
curl -X PUT http://localhost:3000/students/1 \
  -H "Content-Type: application/json" \
  -d '{"name":"John Updated"}'
```

DELETE request

```bash
curl -X DELETE http://localhost:3000/students/1
```

Everything works the same as before

But the code is much cleaner

---

## Practice Exercises

### Exercise 1

Create a new Express server

Add a route

```text
GET /welcome
```

Return the message "Welcome to Express"

### Exercise 2

Create a products array

Implement GET /products to return all products

### Exercise 3

Add a POST route to add new products

Use req.body to get the data

### Exercise 4

Add a route parameter

```text
GET /users/:userId
```

Return the userId from the URL

Example: /users/5 should return { userId: 5 }

---

## Interview Questions

### What is Express.js

Express.js is a framework for Node.js that helps build web applications

### Why do we use Express

Express makes it easier to build web servers with less code

### What does app.use(express.json()) do

It tells Express to automatically parse JSON from incoming request bodies

### How do you get data from URL parameters

Using req.params

Example: req.params.id

### How do you get data from query strings

Using req.query

Example: req.query.name

### What is the difference between res.send() and res.json()

res.send() can send text, HTML, or JSON
res.json() specifically sends JSON and sets the correct header

---

## Summary

In this session, you learned

* What Express.js is
* Why Express is better than the HTTP module
* How to install Express
* How to create routes in Express
* How to send JSON responses
* How to handle GET, POST, PUT, DELETE requests
* How to use URL parameters
* How to use req.body for POST data
* How to use req.params for route parameters

You have rebuilt your student API with cleaner code

---