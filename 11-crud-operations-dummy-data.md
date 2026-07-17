## Table of Contents

* [What is CRUD](#what-is-crud)
* [Understanding REST API](#understanding-rest-api)
* [Setting Up the Project](#setting-up-the-project)
* [Our Dummy Data](#our-dummy-data)
* [GET - Get All Students](#get---get-all-students)
* [GET - Get One Student](#get---get-one-student)
* [POST - Add a New Student](#post---add-a-new-student)
* [PUT - Update a Student](#put---update-a-student)
* [DELETE - Remove a Student](#delete---remove-a-student)
* [Complete Code](#complete-code)
* [Testing Your API](#testing-your-api)
* [Practice Exercises](#practice-exercises)
* [Interview Questions](#interview-questions)
* [Summary](#summary)

---

## What is CRUD

CRUD is a word we use in backend development

It stands for

```text
C -> Create
R -> Read
U -> Update
D -> Delete
```

Think of a student management system

```text
Create -> Add a new student to the list
Read   -> See all students or one student
Update -> Change student information
Delete -> Remove a student from the list
```

Every backend application needs these four operations

---

## Understanding REST API

REST API is just a way to build web applications

We use different URLs for different tasks

```text
/students        -> Work with students
/products        -> Work with products
/users           -> Work with users
```

We also use HTTP methods

```text
GET    -> Get data
POST   -> Send new data
PUT    -> Update existing data
DELETE -> Remove data
```

Our API will look like this

| What we want to do | Method | URL |
| ------------------ | ------ | --- |
| Get all students | GET | /students |
| Get one student | GET | /students/1 |
| Add a student | POST | /students |
| Update a student | PUT | /students/1 |
| Delete a student | DELETE | /students/1 |

The number at the end like /students/1 is the student id

---

## Setting Up the Project

Open your terminal

Create a new folder

```bash
mkdir student-api
```

Go inside the folder

```bash
cd student-api
```

Create a package.json file

```bash
npm init -y
```

Create a file named server.js

Now open server.js in your code editor

First line to import http module

```javascript
const http = require("http");
```

---

## Our Dummy Data

We will store students in an array

An array works like a temporary database

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
  },
  {
    id: 3,
    name: "Mike Johnson",
    age: 21,
    course: "Physics"
  }
];
```

Each student has

```text
id -> A unique number
name -> Student name
age -> Student age
course -> What they are studying
```

We will add, update, and delete from this array

---

## GET - Get All Students

When someone visits http://localhost:3000/students

We want to send back all students

```javascript
if (req.url === "/students" && req.method === "GET") {
  res.statusCode = 200;
  res.setHeader("Content-Type", "application/json");
  res.end(JSON.stringify(students));
}
```

Explanation

```text
req.url === "/students" -> Is the URL /students
req.method === "GET" -> Is the request type GET
res.statusCode = 200 -> Everything is OK
res.setHeader -> Tell browser we are sending JSON
JSON.stringify(students) -> Convert array to JSON string
```

Try visiting http://localhost:3000/students in your browser

You will see all students

---

## GET - Get One Student

When someone visits http://localhost:3000/students/1

We want to send back only the student with id 1

We need to get the id from the URL

Example

```text
/students/1 -> id is 1
/students/2 -> id is 2
/students/3 -> id is 3
```

Here is the code

```javascript
if (req.url.startsWith("/students/") && req.method === "GET") {
  const parts = req.url.split("/");
  const id = parseInt(parts[2]);
  
  const student = students.find(s => s.id === id);
  
  if (student) {
    res.statusCode = 200;
    res.setHeader("Content-Type", "application/json");
    res.end(JSON.stringify(student));
  } else {
    res.statusCode = 404;
    res.end(JSON.stringify({ message: "Student not found" }));
  }
}
```

Let me explain each line

```text
req.url.startsWith("/students/") -> Does URL start with /students/
parts = req.url.split("/") -> Split URL by slash
Example /students/1 becomes ["", "students", "1"]
id = parseInt(parts[2]) -> Get the third item and convert to number
student = students.find(s => s.id === id) -> Find student with matching id
```

If student exists

```text
Send status 200
Send the student data
```

If student does not exist

```text
Send status 404
Send error message
```

---

## POST - Add a New Student

When someone wants to add a new student

They send data in the request body

The code looks a bit different because we need to read the data

```javascript
if (req.url === "/students" && req.method === "POST") {
  let body = "";
  
  req.on("data", (chunk) => {
    body += chunk.toString();
  });
  
  req.on("end", () => {
    const newStudent = JSON.parse(body);
    
    newStudent.id = students.length + 1;
    
    students.push(newStudent);
    
    res.statusCode = 201;
    res.setHeader("Content-Type", "application/json");
    res.end(JSON.stringify(newStudent));
  });
}
```

Explanation

```text
let body = "" -> Create empty string to store incoming data
req.on("data") -> Every time data arrives, add it to body
req.on("end") -> When all data arrives
newStudent = JSON.parse(body) -> Convert JSON string to object
newStudent.id = students.length + 1 -> Create new id
students.push(newStudent) -> Add to our array
res.statusCode = 201 -> Created successfully
```

To test POST, you cannot use browser

You need Postman or curl command

Example request body

```json
{
  "name": "Sarah Wilson",
  "age": 23,
  "course": "Biology"
}
```

---

## PUT - Update a Student

When someone wants to update a student

They send the new data in the request body

We need the student id from the URL

```javascript
if (req.url.startsWith("/students/") && req.method === "PUT") {
  const parts = req.url.split("/");
  const id = parseInt(parts[2]);
  
  let body = "";
  
  req.on("data", (chunk) => {
    body += chunk.toString();
  });
  
  req.on("end", () => {
    const updatedData = JSON.parse(body);
    
    const index = students.findIndex(s => s.id === id);
    
    if (index !== -1) {
      students[index] = { ...students[index], ...updatedData };
      
      res.statusCode = 200;
      res.setHeader("Content-Type", "application/json");
      res.end(JSON.stringify(students[index]));
    } else {
      res.statusCode = 404;
      res.end(JSON.stringify({ message: "Student not found" }));
    }
  });
}
```

Explanation

```text
parts = req.url.split("/") -> Split URL
id = parseInt(parts[2]) -> Get id from URL
body -> Collect incoming data
updatedData -> Convert JSON to object
index = students.findIndex(s => s.id === id) -> Find position of student
```

If student exists

```text
Update the student data
Send back updated student
```

If student does not exist

```text
Send 404 error
```

Example request body to update name

```json
{
  "name": "John Updated"
}
```

---

## DELETE - Remove a Student

When someone wants to delete a student

We need the student id from the URL

```javascript
if (req.url.startsWith("/students/") && req.method === "DELETE") {
  const parts = req.url.split("/");
  const id = parseInt(parts[2]);
  
  const index = students.findIndex(s => s.id === id);
  
  if (index !== -1) {
    students.splice(index, 1);
    
    res.statusCode = 200;
    res.setHeader("Content-Type", "application/json");
    res.end(JSON.stringify({ message: "Student deleted successfully" }));
  } else {
    res.statusCode = 404;
    res.end(JSON.stringify({ message: "Student not found" }));
  }
}
```

Explanation

```text
index = students.findIndex(s => s.id === id) -> Find position of student
students.splice(index, 1) -> Remove from array
Send success message
```

---

## Complete Code

Here is the complete server.js file

```javascript
const http = require("http");

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

const server = http.createServer((req, res) => {
  res.setHeader("Content-Type", "application/json");
  
  // GET all students
  if (req.url === "/students" && req.method === "GET") {
    res.statusCode = 200;
    res.end(JSON.stringify(students));
  }
  
  // GET one student
  else if (req.url.startsWith("/students/") && req.method === "GET") {
    const parts = req.url.split("/");
    const id = parseInt(parts[2]);
    
    const student = students.find(s => s.id === id);
    
    if (student) {
      res.statusCode = 200;
      res.end(JSON.stringify(student));
    } else {
      res.statusCode = 404;
      res.end(JSON.stringify({ message: "Student not found" }));
    }
  }
  
  // POST add new student
  else if (req.url === "/students" && req.method === "POST") {
    let body = "";
    
    req.on("data", (chunk) => {
      body += chunk.toString();
    });
    
    req.on("end", () => {
      const newStudent = JSON.parse(body);
      newStudent.id = students.length + 1;
      students.push(newStudent);
      
      res.statusCode = 201;
      res.end(JSON.stringify(newStudent));
    });
  }
  
  // PUT update student
  else if (req.url.startsWith("/students/") && req.method === "PUT") {
    const parts = req.url.split("/");
    const id = parseInt(parts[2]);
    
    let body = "";
    
    req.on("data", (chunk) => {
      body += chunk.toString();
    });
    
    req.on("end", () => {
      const updatedData = JSON.parse(body);
      const index = students.findIndex(s => s.id === id);
      
      if (index !== -1) {
        students[index] = { ...students[index], ...updatedData };
        
        res.statusCode = 200;
        res.end(JSON.stringify(students[index]));
      } else {
        res.statusCode = 404;
        res.end(JSON.stringify({ message: "Student not found" }));
      }
    });
  }
  
  // DELETE student
  else if (req.url.startsWith("/students/") && req.method === "DELETE") {
    const parts = req.url.split("/");
    const id = parseInt(parts[2]);
    
    const index = students.findIndex(s => s.id === id);
    
    if (index !== -1) {
      students.splice(index, 1);
      
      res.statusCode = 200;
      res.end(JSON.stringify({ message: "Student deleted successfully" }));
    } else {
      res.statusCode = 404;
      res.end(JSON.stringify({ message: "Student not found" }));
    }
  }
  
  // Route not found
  else {
    res.statusCode = 404;
    res.end(JSON.stringify({ message: "Route not found" }));
  }
});

server.listen(3000, () => {
  console.log("Server running on port 3000");
  console.log("Try these URLs");
  console.log("GET    http://localhost:3000/students");
  console.log("GET    http://localhost:3000/students/1");
  console.log("POST   http://localhost:3000/students");
  console.log("PUT    http://localhost:3000/students/1");
  console.log("DELETE http://localhost:3000/students/1");
});
```

---

## Testing Your API

Run the server

```bash
node server.js
```

### Testing GET requests

Open your browser and visit

```text
http://localhost:3000/students
```

You will see all students

```text
http://localhost:3000/students/1
```

You will see student with id 1

### Testing POST, PUT, DELETE

Browser only works for GET requests

For POST, PUT, DELETE you need Postman

Download Postman from https://www.postman.com

Or use curl in terminal

POST request using curl

```bash
curl -X POST http://localhost:3000/students \
  -H "Content-Type: application/json" \
  -d '{"name":"Sarah","age":23,"course":"Biology"}'
```

PUT request using curl

```bash
curl -X PUT http://localhost:3000/students/1 \
  -H "Content-Type: application/json" \
  -d '{"name":"John Updated"}'
```

DELETE request using curl

```bash
curl -X DELETE http://localhost:3000/students/1
```

---

## HTTP Status Codes

Status codes tell the client what happened

| Code | Meaning | When to use |
| ---- | ------- | ----------- |
| 200 | OK | Everything worked |
| 201 | Created | New student was added |
| 404 | Not Found | Student does not exist |
| 500 | Server Error | Something broke on server |

Always use the correct status code

---

## Practice Exercises

### Exercise 1

Create a products API

Each product should have

```text
id
name
price
category
```

Implement all CRUD operations

### Exercise 2

Add validation to POST request

Check if name is empty

If empty, return status 400 with message "Name is required"

### Exercise 3

When getting a student that does not exist

Return status 404 with message "Student not found"

### Exercise 4

Add a new route

```text
GET /students/count
```

Return the total number of students

Example response

```json
{
  "totalStudents": 3
}
```

---

## Interview Questions

### What does CRUD stand for

Create, Read, Update, Delete

### What is the difference between GET and POST

GET retrieves data
POST sends new data

### What is the difference between PUT and POST

POST creates a new resource
PUT updates an existing resource

### What status code does POST return

201 Created

### What status code does GET return

200 OK

### Can we test POST requests in browser

No
Browser only handles GET requests
We need Postman or curl

---

## Summary

In this session, you learned

* What CRUD operations are
* How to create a REST API
* GET to retrieve students
* POST to add new students
* PUT to update students
* DELETE to remove students
* How to use an array as a database
* HTTP status codes
* How to test your API

You have built your first complete backend API

---