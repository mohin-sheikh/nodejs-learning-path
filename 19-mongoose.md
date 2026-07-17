## Table of Contents

* [What is Mongoose](#what-is-mongoose)
* [Why Use Mongoose](#why-use-mongoose)
* [Installing Mongoose](#installing-mongoose)
* [Connecting to MongoDB with Mongoose](#connecting-to-mongodb-with-mongoose)
* [What is a Schema](#what-is-a-schema)
* [Creating a Schema](#creating-a-schema)
* [What is a Model](#what-is-a-model)
* [Creating a Model](#creating-a-model)
* [CRUD Operations with Mongoose](#crud-operations-with-mongoose)
* [Create Operation](#create-operation)
* [Read Operation](#read-operation)
* [Update Operation](#update-operation)
* [Delete Operation](#delete-operation)
* [Schema Validation](#schema-validation)
* [Complete Student API with Mongoose](#complete-student-api-with-mongoose)
* [Practice Exercises](#practice-exercises)
* [Interview Questions](#interview-questions)
* [Summary](#summary)

---

## What is Mongoose

Mongoose is an ODM for MongoDB

ODM stands for Object Data Mapping

It helps you work with MongoDB using JavaScript objects

Think of it like a translator

```text
Your JavaScript code -> Mongoose -> MongoDB
MongoDB -> Mongoose -> JavaScript objects
```

Without Mongoose, we wrote code like this

```javascript
const result = await db.collection("students").insertOne({
  name: "John",
  age: 20
});
```

With Mongoose, we write code like this

```javascript
const student = new Student({
  name: "John",
  age: 20
});
await student.save();
```

Mongoose makes MongoDB feel like working with JavaScript objects

---

## Why Use Mongoose

Benefits of Mongoose

```text
Schemas -> Define how your data should look
Validation -> Automatically check data before saving
Queries -> Easier syntax for finding data
Middleware -> Run code before or after operations
Relationships -> Connect different collections
```

Problems without Mongoose

```text
No structure enforcement
Manual validation required
Complex query syntax
No built-in relationships
```

Problems solved by Mongoose

```text
Schema defines structure
Automatic validation
Simple query methods
Populate for relationships
```

---

## Installing Mongoose

Create a new project

```bash
mkdir mongoose-demo
cd mongoose-demo
npm init -y
```

Install required packages

```bash
npm install express mongoose dotenv
```

Create a .env file

```text
MONGODB_URI=mongodb+srv://yourusername:yourpassword@cluster0.abc123.mongodb.net/
DB_NAME=school
PORT=3000
```

Now we are ready to use Mongoose

---

## Connecting to MongoDB with Mongoose

Connecting with Mongoose is very simple

```javascript
const mongoose = require("mongoose");

mongoose.connect("mongodb+srv://username:password@cluster0.abc123.mongodb.net/school")
  .then(() => {
    console.log("Connected to MongoDB");
  })
  .catch((error) => {
    console.error("Connection failed", error);
  });
```

Using environment variables

```javascript
require("dotenv").config();
const mongoose = require("mongoose");

const MONGODB_URI = process.env.MONGODB_URI;
const DB_NAME = process.env.DB_NAME;

mongoose.connect(`${MONGODB_URI}/${DB_NAME}`)
  .then(() => console.log("Connected to MongoDB"))
  .catch(err => console.error("Connection failed", err));
```

Connection events you can listen to

```javascript
mongoose.connection.on("connected", () => {
  console.log("Mongoose connected to MongoDB");
});

mongoose.connection.on("error", (err) => {
  console.log("Mongoose connection error", err);
});

mongoose.connection.on("disconnected", () => {
  console.log("Mongoose disconnected");
});
```

---

## What is a Schema

A schema defines the structure of your documents

It tells Mongoose what fields a document should have

Think of a schema like a blueprint

```text
Blueprint tells you
- What rooms a house has
- What size each room is
- What materials to use

Schema tells you
- What fields a document has
- What type each field is
- What validation is required
```

Example of a student schema

```javascript
const studentSchema = new mongoose.Schema({
  name: String,
  age: Number,
  course: String,
  email: String
});
```

This means every student document must have these fields

---

## Creating a Schema

Let us create a proper schema for a student

```javascript
const studentSchema = new mongoose.Schema({
  name: {
    type: String,
    required: true
  },
  age: {
    type: Number,
    required: true,
    min: 18,
    max: 60
  },
  course: {
    type: String,
    default: "Not specified"
  },
  email: {
    type: String,
    required: true,
    unique: true
  },
  createdAt: {
    type: Date,
    default: Date.now
  }
});
```

Schema field options

| Option | Meaning |
| ------ | ------- |
| type | Data type (String, Number, Date, Boolean) |
| required | Field must be provided |
| default | Default value if not provided |
| unique | No two documents can have same value |
| min | Minimum value for numbers |
| max | Maximum value for numbers |
| lowercase | Convert to lowercase |
| uppercase | Convert to uppercase |
| trim | Remove whitespace |

---

## What is a Model

A model is a wrapper around a schema

It gives you methods to work with a specific collection

Think of it like this

```text
Schema = Blueprint
Model = Factory that creates documents from the blueprint
```

One schema can have one model

The model name should be singular and start with capital letter

```javascript
const Student = mongoose.model("Student", studentSchema);
```

Mongoose will automatically

```text
Use model name "Student"
Create collection name "students" (lowercase and plural)
```

Now you can use Student to perform CRUD operations

---

## Creating a Model

Create a folder named models

Create a file models/Student.js

```javascript
const mongoose = require("mongoose");

const studentSchema = new mongoose.Schema({
  name: {
    type: String,
    required: [true, "Name is required"],
    trim: true
  },
  age: {
    type: Number,
    required: [true, "Age is required"],
    min: [18, "Age must be at least 18"],
    max: [60, "Age cannot exceed 60"]
  },
  course: {
    type: String,
    default: "Not specified"
  },
  email: {
    type: String,
    required: [true, "Email is required"],
    unique: true,
    lowercase: true,
    trim: true
  },
  createdAt: {
    type: Date,
    default: Date.now
  }
});

module.exports = mongoose.model("Student", studentSchema);
```

Now you can import and use this model anywhere

---

## CRUD Operations with Mongoose

Mongoose makes CRUD operations very simple

Let me show you each operation

---

## Create Operation

Method 1 - Create and save

```javascript
const student = new Student({
  name: "John Doe",
  age: 20,
  course: "Computer Science",
  email: "john@example.com"
});

const savedStudent = await student.save();
console.log(savedStudent);
```

Method 2 - Create directly

```javascript
const student = await Student.create({
  name: "John Doe",
  age: 20,
  course: "Computer Science",
  email: "john@example.com"
});
```

Method 3 - Create multiple

```javascript
const students = await Student.insertMany([
  { name: "John", age: 20, email: "john@example.com" },
  { name: "Jane", age: 22, email: "jane@example.com" }
]);
```

---

## Read Operation

Find all documents

```javascript
const allStudents = await Student.find();
```

Find with filter

```javascript
const csStudents = await Student.find({ course: "Computer Science" });
```

Find one document

```javascript
const student = await Student.findOne({ name: "John Doe" });
```

Find by id

```javascript
const student = await Student.findById("507f1f77bcf86cd799439011");
```

Select specific fields

```javascript
const students = await Student.find().select("name email");
```

Limit and skip (pagination)

```javascript
const students = await Student.find()
  .limit(10)
  .skip(20);
```

Sort

```javascript
// Ascending
const students = await Student.find().sort("name");

// Descending
const students = await Student.find().sort("-age");
```

Count documents

```javascript
const count = await Student.countDocuments({ course: "Computer Science" });
```

---

## Update Operation

Update one document

```javascript
const result = await Student.updateOne(
  { name: "John Doe" },
  { $set: { age: 21 } }
);
```

Update multiple documents

```javascript
const result = await Student.updateMany(
  { course: "Computer Science" },
  { $set: { department: "Engineering" } }
);
```

Find and update (returns updated document)

```javascript
const student = await Student.findByIdAndUpdate(
  "507f1f77bcf86cd799439011",
  { age: 21 },
  { new: true }  // Returns updated document
);
```

---

## Delete Operation

Delete one document

```javascript
const result = await Student.deleteOne({ name: "John Doe" });
```

Delete by id

```javascript
const result = await Student.findByIdAndDelete("507f1f77bcf86cd799439011");
```

Delete multiple documents

```javascript
const result = await Student.deleteMany({ age: { $lt: 18 } });
```

Delete all documents

```javascript
const result = await Student.deleteMany({});
```

---

## Schema Validation

Mongoose automatically validates data before saving

Built-in validators

```javascript
const studentSchema = new mongoose.Schema({
  name: {
    type: String,
    required: [true, "Name is required"],
    minlength: [2, "Name must be at least 2 characters"],
    maxlength: [50, "Name cannot exceed 50 characters"]
  },
  age: {
    type: Number,
    required: true,
    min: 18,
    max: 60
  },
  email: {
    type: String,
    required: true,
    unique: true,
    match: [/^\S+@\S+\.\S+$/, "Please enter a valid email"]
  },
  course: {
    type: String,
    enum: {
      values: ["Computer Science", "Mathematics", "Physics", "Biology"],
      message: "Course must be one of the listed options"
    }
  }
});
```

Custom validator

```javascript
const studentSchema = new mongoose.Schema({
  phoneNumber: {
    type: String,
    validate: {
      validator: function(v) {
        return /\d{10}/.test(v);
      },
      message: "Phone number must be 10 digits"
    }
  }
});
```

---

## Complete Student API with Mongoose

Project structure

```text
mongoose-api/
├── models/
│   └── Student.js
├── .env
├── server.js
└── package.json
```

models/Student.js

```javascript
const mongoose = require("mongoose");

const studentSchema = new mongoose.Schema({
  name: {
    type: String,
    required: [true, "Name is required"],
    trim: true,
    minlength: [2, "Name must be at least 2 characters"]
  },
  age: {
    type: Number,
    required: [true, "Age is required"],
    min: [18, "Age must be at least 18"],
    max: [60, "Age cannot exceed 60"]
  },
  course: {
    type: String,
    default: "Not specified"
  },
  email: {
    type: String,
    required: [true, "Email is required"],
    unique: true,
    lowercase: true,
    match: [/^\S+@\S+\.\S+$/, "Please enter a valid email"]
  }
}, {
  timestamps: true  // Adds createdAt and updatedAt automatically
});

module.exports = mongoose.model("Student", studentSchema);
```

server.js

```javascript
require("dotenv").config();
const express = require("express");
const mongoose = require("mongoose");
const Student = require("./models/Student");

const app = express();
app.use(express.json());

// Connect to MongoDB
mongoose.connect(`${process.env.MONGODB_URI}/${process.env.DB_NAME}`)
  .then(() => console.log("Connected to MongoDB"))
  .catch(err => console.error("Connection failed", err));

// CREATE - Add a student
app.post("/students", async (req, res) => {
  try {
    const student = await Student.create(req.body);
    res.status(201).json({
      success: true,
      data: student
    });
  } catch (error) {
    res.status(400).json({
      success: false,
      message: error.message
    });
  }
});

// READ - Get all students
app.get("/students", async (req, res) => {
  try {
    const students = await Student.find();
    res.json({
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
});

// READ - Get one student
app.get("/students/:id", async (req, res) => {
  try {
    const student = await Student.findById(req.params.id);
    
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
  } catch (error) {
    res.status(500).json({
      success: false,
      message: error.message
    });
  }
});

// UPDATE - Update a student
app.put("/students/:id", async (req, res) => {
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
    
    res.json({
      success: true,
      data: student
    });
  } catch (error) {
    res.status(400).json({
      success: false,
      message: error.message
    });
  }
});

// DELETE - Delete a student
app.delete("/students/:id", async (req, res) => {
  try {
    const student = await Student.findByIdAndDelete(req.params.id);
    
    if (!student) {
      return res.status(404).json({
        success: false,
        message: "Student not found"
      });
    }
    
    res.json({
      success: true,
      message: "Student deleted successfully"
    });
  } catch (error) {
    res.status(500).json({
      success: false,
      message: error.message
    });
  }
});

const PORT = process.env.PORT || 3000;
app.listen(PORT, () => {
  console.log(`Server running on port ${PORT}`);
});
```

---

## Practice Exercises

### Exercise 1

Create a Product model with

```text
name (required, min 3 characters)
price (required, min 0)
category (required)
stock (default 0)
```

### Exercise 2

Add validation to prevent duplicate product names

### Exercise 3

Create a route to find products by category

GET /products/category/:category

### Exercise 4

Create a route to find products with low stock

GET /products/low-stock?lessThan=10

### Exercise 5

Add timestamps to your Product schema

Use { timestamps: true }

---

## Interview Questions

### What is Mongoose

Mongoose is an ODM (Object Data Mapping) library for MongoDB and Node.js

### What is the difference between a Schema and a Model

Schema defines the structure of documents
Model creates an interface to interact with the collection

### Why do we use schemas in Mongoose

To define the structure, validation, and default values for documents

### What is the purpose of timestamps option

It automatically adds createdAt and updatedAt fields to every document

### What does { new: true } do in findByIdAndUpdate

It returns the updated document instead of the original

### What is the difference between create and save

create creates and saves in one step
save requires creating an instance first then saving

### How does Mongoose handle validation

It automatically validates data against the schema before saving

---

## Summary

In this session, you learned

* What Mongoose is
* Why we use Mongoose
* How to connect to MongoDB with Mongoose
* What schemas are and how to create them
* What models are and how to create them
* CRUD operations with Mongoose
* Schema validation
* Built-in validators like required, min, max
* How to build a complete API with Mongoose

Mongoose makes working with MongoDB much easier and more structured

---