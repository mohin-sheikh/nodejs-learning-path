## Table of Contents

* [Review of CRUD](#review-of-crud)
* [Setting Up the Project](#setting-up-the-project)
* [Connecting to MongoDB](#connecting-to-mongodb)
* [Create Operation - Insert](#create-operation---insert)
* [Read Operation - Find](#read-operation---find)
* [Update Operation - Update](#read-operation---find)
* [Delete Operation - Delete](#delete-operation---delete)
* [Query Operators](#query-operators)
* [Complete CRUD Example](#complete-crud-example)
* [Practice Exercises](#practice-exercises)
* [Interview Questions](#interview-questions)
* [Summary](#summary)

---

## Review of CRUD

CRUD is the same whether using an array or a database

| Operation | MongoDB Method | HTTP Method |
| --------- | -------------- | ----------- |
| Create | insertOne(), insertMany() | POST |
| Read | find(), findOne() | GET |
| Update | updateOne(), updateMany() | PUT |
| Delete | deleteOne(), deleteMany() | DELETE |

The difference is that MongoDB saves data permanently

No data loss when server restarts

---

## Setting Up the Project

Create a new project

```bash
mkdir mongodb-crud
cd mongodb-crud
npm init -y
```

Install required packages

```bash
npm install express mongodb dotenv
```

Create a .env file

```text
MONGODB_URI=mongodb+srv://yourusername:yourpassword@cluster0.abc123.mongodb.net/
DB_NAME=school
PORT=3000
```

Create a file named server.js

We will build our student API with MongoDB instead of an array

---

## Connecting to MongoDB

First, let us create a database connection

Create a file named db.js

```javascript
const { MongoClient } = require("mongodb");

const uri = process.env.MONGODB_URI;
const dbName = process.env.DB_NAME;

let db;

async function connectToDB() {
  try {
    const client = new MongoClient(uri);
    await client.connect();
    console.log("Connected to MongoDB");
    
    db = client.db(dbName);
    return db;
  } catch (error) {
    console.error("Database connection failed", error);
    process.exit(1);
  }
}

function getDB() {
  if (!db) {
    throw new Error("Database not connected. Call connectToDB first");
  }
  return db;
}

module.exports = { connectToDB, getDB };
```

Now in server.js

```javascript
require("dotenv").config();
const express = require("express");
const { connectToDB, getDB } = require("./db");

const app = express();
app.use(express.json());

// Connect to database before starting server
async function startServer() {
  await connectToDB();
  
  app.listen(process.env.PORT || 3000, () => {
    console.log(`Server running on port ${process.env.PORT || 3000}`);
  });
}

startServer();
```

---

## Create Operation - Insert

POST /students - Add a new student

```javascript
app.post("/students", async (req, res) => {
  try {
    const db = getDB();
    const studentsCollection = db.collection("students");
    
    const { name, age, course, email } = req.body;
    
    // Validation
    if (!name || !age || !email) {
      return res.status(400).json({
        success: false,
        message: "Name, age, and email are required"
      });
    }
    
    const newStudent = {
      name: name,
      age: age,
      course: course || "Not specified",
      email: email,
      createdAt: new Date()
    };
    
    const result = await studentsCollection.insertOne(newStudent);
    
    res.status(201).json({
      success: true,
      data: {
        id: result.insertedId,
        ...newStudent
      }
    });
    
  } catch (error) {
    res.status(500).json({
      success: false,
      message: "Failed to create student"
    });
  }
});
```

Insert multiple students at once

```javascript
app.post("/students/bulk", async (req, res) => {
  try {
    const db = getDB();
    const studentsCollection = db.collection("students");
    
    const students = req.body.students;
    
    if (!students || !students.length) {
      return res.status(400).json({
        success: false,
        message: "Students array is required"
      });
    }
    
    const result = await studentsCollection.insertMany(students);
    
    res.status(201).json({
      success: true,
      insertedCount: result.insertedCount
    });
    
  } catch (error) {
    res.status(500).json({
      success: false,
      message: "Failed to create students"
    });
  }
});
```

---

## Read Operation - Find

GET /students - Get all students

```javascript
app.get("/students", async (req, res) => {
  try {
    const db = getDB();
    const studentsCollection = db.collection("students");
    
    const students = await studentsCollection.find({}).toArray();
    
    res.status(200).json({
      success: true,
      count: students.length,
      data: students
    });
    
  } catch (error) {
    res.status(500).json({
      success: false,
      message: "Failed to fetch students"
    });
  }
});
```

GET /students/:id - Get one student

```javascript
const { ObjectId } = require("mongodb");

app.get("/students/:id", async (req, res) => {
  try {
    const db = getDB();
    const studentsCollection = db.collection("students");
    
    const id = req.params.id;
    
    // Check if id is valid
    if (!ObjectId.isValid(id)) {
      return res.status(400).json({
        success: false,
        message: "Invalid student id"
      });
    }
    
    const student = await studentsCollection.findOne({ 
      _id: new ObjectId(id) 
    });
    
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
      message: "Failed to fetch student"
    });
  }
});
```

GET with filters

```javascript
app.get("/students/filter/by-age", async (req, res) => {
  try {
    const db = getDB();
    const studentsCollection = db.collection("students");
    
    const minAge = parseInt(req.query.minAge);
    const maxAge = parseInt(req.query.maxAge);
    
    let filter = {};
    
    if (minAge && maxAge) {
      filter.age = { $gte: minAge, $lte: maxAge };
    } else if (minAge) {
      filter.age = { $gte: minAge };
    } else if (maxAge) {
      filter.age = { $lte: maxAge };
    }
    
    const students = await studentsCollection.find(filter).toArray();
    
    res.status(200).json({
      success: true,
      count: students.length,
      data: students
    });
    
  } catch (error) {
    res.status(500).json({
      success: false,
      message: "Failed to fetch students"
    });
  }
});
```

---

## Update Operation - Update

PUT /students/:id - Update a student

```javascript
app.put("/students/:id", async (req, res) => {
  try {
    const db = getDB();
    const studentsCollection = db.collection("students");
    
    const id = req.params.id;
    
    if (!ObjectId.isValid(id)) {
      return res.status(400).json({
        success: false,
        message: "Invalid student id"
      });
    }
    
    const { name, age, course, email } = req.body;
    
    const updateData = {};
    if (name) updateData.name = name;
    if (age) updateData.age = age;
    if (course) updateData.course = course;
    if (email) updateData.email = email;
    updateData.updatedAt = new Date();
    
    const result = await studentsCollection.updateOne(
      { _id: new ObjectId(id) },
      { $set: updateData }
    );
    
    if (result.matchedCount === 0) {
      return res.status(404).json({
        success: false,
        message: "Student not found"
      });
    }
    
    const updatedStudent = await studentsCollection.findOne({ 
      _id: new ObjectId(id) 
    });
    
    res.status(200).json({
      success: true,
      data: updatedStudent
    });
    
  } catch (error) {
    res.status(500).json({
      success: false,
      message: "Failed to update student"
    });
  }
});
```

Update multiple students

```javascript
app.put("/students/course/update", async (req, res) => {
  try {
    const db = getDB();
    const studentsCollection = db.collection("students");
    
    const { oldCourse, newCourse } = req.body;
    
    const result = await studentsCollection.updateMany(
      { course: oldCourse },
      { $set: { course: newCourse } }
    );
    
    res.status(200).json({
      success: true,
      modifiedCount: result.modifiedCount
    });
    
  } catch (error) {
    res.status(500).json({
      success: false,
      message: "Failed to update students"
    });
  }
});
```

---

## Delete Operation - Delete

DELETE /students/:id - Delete a student

```javascript
app.delete("/students/:id", async (req, res) => {
  try {
    const db = getDB();
    const studentsCollection = db.collection("students");
    
    const id = req.params.id;
    
    if (!ObjectId.isValid(id)) {
      return res.status(400).json({
        success: false,
        message: "Invalid student id"
      });
    }
    
    const result = await studentsCollection.deleteOne({ 
      _id: new ObjectId(id) 
    });
    
    if (result.deletedCount === 0) {
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
      message: "Failed to delete student"
    });
  }
});
```

Delete multiple students

```javascript
app.delete("/students/course/:courseName", async (req, res) => {
  try {
    const db = getDB();
    const studentsCollection = db.collection("students");
    
    const courseName = req.params.courseName;
    
    const result = await studentsCollection.deleteMany({ 
      course: courseName 
    });
    
    res.status(200).json({
      success: true,
      deletedCount: result.deletedCount,
      message: `${result.deletedCount} students deleted`
    });
    
  } catch (error) {
    res.status(500).json({
      success: false,
      message: "Failed to delete students"
    });
  }
});
```

---

## Query Operators

MongoDB provides operators for complex queries

Comparison operators

| Operator | Meaning | Example |
| -------- | ------- | ------- |
| $eq | Equal | { age: { $eq: 20 } } |
| $ne | Not equal | { age: { $ne: 20 } } |
| $gt | Greater than | { age: { $gt: 20 } } |
| $gte | Greater than or equal | { age: { $gte: 20 } } |
| $lt | Less than | { age: { $lt: 20 } } |
| $lte | Less than or equal | { age: { $lte: 20 } } |
| $in | In array | { course: { $in: ["CS", "Math"] } } |
| $nin | Not in array | { course: { $nin: ["CS", "Math"] } } |

Logical operators

| Operator | Meaning | Example |
| -------- | ------- | ------- |
| $and | All conditions true | { $and: [{age: 20}, {course: "CS"}] } |
| $or | Any condition true | { $or: [{age: 20}, {age: 22}] } |
| $nor | No condition true | { $nor: [{age: 20}] } |
| $not | Negates condition | { age: { $not: { $gt: 20 } } } |

Example routes with operators

```javascript
// Students older than 20
app.get("/students/age/above/:age", async (req, res) => {
  const db = getDB();
  const age = parseInt(req.params.age);
  
  const students = await db.collection("students")
    .find({ age: { $gt: age } })
    .toArray();
    
  res.json(students);
});

// Students in CS or Math
app.get("/students/courses/multiple", async (req, res) => {
  const db = getDB();
  
  const students = await db.collection("students")
    .find({ course: { $in: ["Computer Science", "Mathematics"] } })
    .toArray();
    
  res.json(students);
});

// Students who are 20 AND in CS
app.get("/students/condition/and", async (req, res) => {
  const db = getDB();
  
  const students = await db.collection("students")
    .find({
      $and: [
        { age: 20 },
        { course: "Computer Science" }
      ]
    })
    .toArray();
    
  res.json(students);
});
```

---

## Complete CRUD Example

Here is the complete server.js with all CRUD operations

```javascript
require("dotenv").config();
const express = require("express");
const { MongoClient, ObjectId } = require("mongodb");

const app = express();
app.use(express.json());

let db;

async function connectToDB() {
  const client = new MongoClient(process.env.MONGODB_URI);
  await client.connect();
  console.log("Connected to MongoDB");
  db = client.db(process.env.DB_NAME);
}

// CREATE - Add a student
app.post("/students", async (req, res) => {
  try {
    const { name, age, course, email } = req.body;
    
    if (!name || !age || !email) {
      return res.status(400).json({ error: "Name, age, and email are required" });
    }
    
    const result = await db.collection("students").insertOne({
      name, age, course: course || "Not specified", email, createdAt: new Date()
    });
    
    res.status(201).json({ id: result.insertedId, name, age, course, email });
  } catch (error) {
    res.status(500).json({ error: "Failed to create student" });
  }
});

// READ - Get all students
app.get("/students", async (req, res) => {
  try {
    const students = await db.collection("students").find({}).toArray();
    res.json(students);
  } catch (error) {
    res.status(500).json({ error: "Failed to fetch students" });
  }
});

// READ - Get one student
app.get("/students/:id", async (req, res) => {
  try {
    const id = req.params.id;
    
    if (!ObjectId.isValid(id)) {
      return res.status(400).json({ error: "Invalid id" });
    }
    
    const student = await db.collection("students").findOne({ _id: new ObjectId(id) });
    
    if (!student) {
      return res.status(404).json({ error: "Student not found" });
    }
    
    res.json(student);
  } catch (error) {
    res.status(500).json({ error: "Failed to fetch student" });
  }
});

// UPDATE - Update a student
app.put("/students/:id", async (req, res) => {
  try {
    const id = req.params.id;
    const { name, age, course, email } = req.body;
    
    if (!ObjectId.isValid(id)) {
      return res.status(400).json({ error: "Invalid id" });
    }
    
    const updateData = {};
    if (name) updateData.name = name;
    if (age) updateData.age = age;
    if (course) updateData.course = course;
    if (email) updateData.email = email;
    updateData.updatedAt = new Date();
    
    const result = await db.collection("students").updateOne(
      { _id: new ObjectId(id) },
      { $set: updateData }
    );
    
    if (result.matchedCount === 0) {
      return res.status(404).json({ error: "Student not found" });
    }
    
    const updatedStudent = await db.collection("students").findOne({ _id: new ObjectId(id) });
    res.json(updatedStudent);
  } catch (error) {
    res.status(500).json({ error: "Failed to update student" });
  }
});

// DELETE - Delete a student
app.delete("/students/:id", async (req, res) => {
  try {
    const id = req.params.id;
    
    if (!ObjectId.isValid(id)) {
      return res.status(400).json({ error: "Invalid id" });
    }
    
    const result = await db.collection("students").deleteOne({ _id: new ObjectId(id) });
    
    if (result.deletedCount === 0) {
      return res.status(404).json({ error: "Student not found" });
    }
    
    res.json({ message: "Student deleted successfully" });
  } catch (error) {
    res.status(500).json({ error: "Failed to delete student" });
  }
});

async function startServer() {
  await connectToDB();
  app.listen(process.env.PORT || 3000, () => {
    console.log(`Server running on port ${process.env.PORT || 3000}`);
  });
}

startServer();
```

---

## Practice Exercises

### Exercise 1

Create a products API with MongoDB

Each product should have

```text
name
price
category
stock
```

Implement all CRUD operations

### Exercise 2

Add a route to find products by category

GET /products/category/:categoryName

### Exercise 3

Add a route to find products within a price range

GET /products/price-range?min=100&max=500

### Exercise 4

Add a route to update stock quantity

PATCH /products/:id/stock

Send { quantity: 50 } in the body

### Exercise 5

Add validation to prevent duplicate emails in student collection

Return 400 if email already exists

---

## Interview Questions

### What is the difference between insertOne and insertMany

insertOne inserts a single document
insertMany inserts multiple documents at once

### What is ObjectId in MongoDB

ObjectId is a unique identifier automatically created for each document

### Why do we need to check ObjectId.isValid

To make sure the id format is correct before trying to use it

### What does the $set operator do

$set updates only the fields you specify without affecting other fields

### What is the difference between updateOne and updateMany

updateOne updates the first matching document
updateMany updates all matching documents

### What does the $gt operator mean

$gt means greater than

### How do you find documents with age between 20 and 30

{ age: { $gte: 20, $lte: 30 } }

---

## Summary

In this session, you learned

* How to perform CRUD operations with MongoDB
* Create - insertOne and insertMany
* Read - find and findOne
* Update - updateOne and updateMany with $set
* Delete - deleteOne and deleteMany
* How to use ObjectId for document ids
* Query operators like $gt, $lt, $in, $and, $or
* How to build a complete REST API with MongoDB

Your data now persists permanently in the database

---