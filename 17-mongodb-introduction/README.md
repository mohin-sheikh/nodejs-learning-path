## Table of Contents

* [What is MongoDB](#what-is-mongodb)
* [SQL vs NoSQL](#sql-vs-nosql)
* [Important MongoDB Terms](#important-mongodb-terms)
* [How MongoDB Stores Data](#how-mongodb-stores-data)
* [Installing MongoDB](#installing-mongodb)
* [MongoDB Atlas Cloud Setup](#mongodb-atlas-cloud-setup)
* [Connecting to MongoDB](#connecting-to-mongodb)
* [MongoDB Compass](#mongodb-compass)
* [Basic MongoDB Commands](#basic-mongodb-commands)
* [Creating a Database](#creating-a-database)
* [Creating a Collection](#creating-a-collection)
* [Inserting Documents](#inserting-documents)
* [Finding Documents](#finding-documents)
* [Updating Documents](#updating-documents)
* [Deleting Documents](#deleting-documents)
* [Practice Exercises](#practice-exercises)
* [Interview Questions](#interview-questions)
* [Summary](#summary)

---

## What is MongoDB

MongoDB is a database

A database stores your application data permanently

Until now we stored data in an array

```javascript
let students = []; // Data lost when server restarts
```

Problems with array storage

```text
Data lost when server restarts
Cannot share data between multiple servers
No search features
No filtering
Slow for large amounts of data
```

MongoDB solves these problems

```text
Data persists even after server restart
Multiple servers can access same data
Powerful search and filtering
Fast even with millions of records
```

MongoDB is a NoSQL database

It stores data in JSON-like format called BSON

---

## SQL vs NoSQL

Traditional databases use SQL (Structured Query Language)

Examples of SQL databases

```text
MySQL
PostgreSQL
SQL Server
Oracle
```

MongoDB uses NoSQL (Not Only SQL)

Examples of NoSQL databases

```text
MongoDB
Firebase
Cassandra
Redis
```

Main differences

| Feature | SQL Database | MongoDB |
| ------- | ------------ | ------- |
| Data format | Tables with rows and columns | Documents in JSON format |
| Schema | Fixed structure | Flexible structure |
| Relationships | Uses foreign keys | Embedded documents |
| Scaling | Vertical (bigger server) | Horizontal (more servers) |

Think of it like this

```text
SQL is like an Excel spreadsheet
Rows and columns, strict format

MongoDB is like a folder of JSON files
Each file can be different, flexible format
```

---

## Important MongoDB Terms

You need to learn these terms

| SQL Term | MongoDB Term | Description |
| -------- | ------------ | ----------- |
| Database | Database | Container for collections |
| Table | Collection | Group of documents |
| Row | Document | Single record |
| Column | Field | Key-value pair |
| Primary Key | _id | Unique identifier |

Example of a student in SQL

```sql
Table: students
| id | name | age | course |
| 1  | John | 20  | CS     |
```

Same student in MongoDB

```json
{
  "_id": 1,
  "name": "John",
  "age": 20,
  "course": "CS"
}
```

---

## How MongoDB Stores Data

MongoDB stores data as documents

A document is like a JavaScript object

```json
{
  "_id": ObjectId("507f1f77bcf86cd799439011"),
  "name": "John Doe",
  "age": 20,
  "course": "Computer Science",
  "email": "john@example.com",
  "address": {
    "city": "New York",
    "zip": "10001"
  },
  "hobbies": ["reading", "coding", "gaming"]
}
```

Features of MongoDB documents

```text
Can have nested objects (address)
Can have arrays (hobbies)
Each document can have different fields
_id is automatically created
```

Multiple documents in a collection

```json
// Collection: students

// Document 1
{
  "_id": 1,
  "name": "John",
  "age": 20
}

// Document 2
{
  "_id": 2,
  "name": "Jane",
  "age": 22,
  "email": "jane@example.com"
}

// Document 3
{
  "_id": 3,
  "name": "Mike",
  "course": "Physics"
}
```

Notice each document can have different fields

MongoDB does not enforce a fixed structure

---

## Installing MongoDB

There are two ways to use MongoDB

Option 1 - Install locally on your computer

Option 2 - Use MongoDB Atlas (cloud)

For beginners, MongoDB Atlas is easier

No installation needed

Free tier available

Works on any computer

Let us use MongoDB Atlas

---

## MongoDB Atlas Cloud Setup

Step 1 - Go to mongodb.com/atlas

Step 2 - Sign up for a free account

Step 3 - Create a new cluster

Choose the FREE tier (M0)

Step 4 - Choose your cloud provider

Any provider is fine (AWS, Google, Azure)

Choose the region closest to you

Step 5 - Click Create Cluster

Wait 1-3 minutes for cluster to be ready

Step 6 - Create a database user

```text
Username: admin
Password: yourpassword (save this)
```

Step 7 - Add your IP address

Click "Add Your Current IP Address"

This allows your computer to connect

Step 8 - Get your connection string

Click Connect

Click "Connect your application"

Copy the connection string

It looks like this

```text
mongodb+srv://admin:<password>@cluster0.abc123.mongodb.net/
```

Replace <password> with your actual password

Save this connection string

You will need it later

---

## Connecting to MongoDB

We need a MongoDB driver to connect from Node.js

Install the MongoDB package

```bash
npm install mongodb
```

Basic connection code

```javascript
const { MongoClient } = require("mongodb");

// Your connection string from Atlas
const url = "mongodb+srv://admin:yourpassword@cluster0.abc123.mongodb.net/";

// Create a new client
const client = new MongoClient(url);

async function connect() {
  try {
    await client.connect();
    console.log("Connected to MongoDB");
    
    // Select database
    const db = client.db("school");
    
    // Select collection
    const collection = db.collection("students");
    
    // Do operations here
    
  } catch (error) {
    console.error("Connection failed", error);
  } finally {
    await client.close();
  }
}

connect();
```

---

## MongoDB Compass

MongoDB Compass is a graphical user interface for MongoDB

It lets you see your data without writing code

Download from mongodb.com/products/compass

It is free

Once connected, you can

```text
See all databases
See all collections
View documents
Add, edit, delete documents
Run queries
```

This is very helpful for beginners

You can see what your code is doing

---

## Basic MongoDB Commands

Once connected, here are common operations

Select a database

```javascript
const db = client.db("school");
```

If database does not exist, MongoDB creates it

Select a collection

```javascript
const students = db.collection("students");
```

If collection does not exist, MongoDB creates it

Now you can perform CRUD operations

---

## Creating a Database

In MongoDB, you do not explicitly create a database

You just start using it

```javascript
// This creates a database named "school" if it doesn't exist
const db = client.db("school");
```

No create command needed

MongoDB creates it when you first insert data

---

## Creating a Collection

Same as database

You do not explicitly create a collection

```javascript
// This creates a collection named "students" if it doesn't exist
const collection = db.collection("students");
```

MongoDB creates the collection when you first insert a document

---

## Inserting Documents

Insert one document

```javascript
const result = await collection.insertOne({
  name: "John Doe",
  age: 20,
  course: "Computer Science",
  email: "john@example.com"
});

console.log("Inserted id:", result.insertedId);
```

Insert multiple documents

```javascript
const result = await collection.insertMany([
  {
    name: "Jane Smith",
    age: 22,
    course: "Mathematics",
    email: "jane@example.com"
  },
  {
    name: "Mike Johnson",
    age: 21,
    course: "Physics",
    email: "mike@example.com"
  }
]);

console.log("Inserted count:", result.insertedCount);
```

---

## Finding Documents

Find all documents

```javascript
const allStudents = await collection.find({}).toArray();
console.log(allStudents);
```

Find documents with filter

```javascript
// Find students with age 20
const result = await collection.find({ age: 20 }).toArray();

// Find students in Computer Science course
const csStudents = await collection.find({ course: "Computer Science" }).toArray();

// Find one student by name
const john = await collection.findOne({ name: "John Doe" });
```

Find with multiple conditions

```javascript
// Age greater than 20
const result = await collection.find({ age: { $gt: 20 } }).toArray();

// Age less than or equal to 21
const result = await collection.find({ age: { $lte: 21 } }).toArray();

// Course is Computer Science AND age is 20
const result = await collection.find({ 
  course: "Computer Science", 
  age: 20 
}).toArray();
```

---

## Updating Documents

Update one document

```javascript
const result = await collection.updateOne(
  { name: "John Doe" },  // Find document
  { $set: { age: 21 } }  // Update age
);

console.log("Modified count:", result.modifiedCount);
```

Update multiple documents

```javascript
const result = await collection.updateMany(
  { course: "Computer Science" },  // Find all CS students
  { $set: { department: "Engineering" } }  // Add department field
);
```

Upsert (update or insert if not exists)

```javascript
const result = await collection.updateOne(
  { name: "Sarah" },
  { $set: { age: 23, course: "Biology" } },
  { upsert: true }  // Create if not exists
);
```

---

## Deleting Documents

Delete one document

```javascript
const result = await collection.deleteOne({ name: "John Doe" });
console.log("Deleted count:", result.deletedCount);
```

Delete multiple documents

```javascript
const result = await collection.deleteMany({ age: { $lt: 18 } });
console.log("Deleted count:", result.deletedCount);
```

Delete all documents in a collection

```javascript
const result = await collection.deleteMany({});
```

Drop entire collection

```javascript
await collection.drop();
```

---

## Complete Example

```javascript
const { MongoClient } = require("mongodb");

// Connection URL
const url = "mongodb+srv://admin:yourpassword@cluster0.abc123.mongodb.net/";

// Database Name
const dbName = "school";

async function main() {
  const client = new MongoClient(url);
  
  try {
    // Connect to MongoDB
    await client.connect();
    console.log("Connected to MongoDB");
    
    // Get database
    const db = client.db(dbName);
    
    // Get collection
    const students = db.collection("students");
    
    // INSERT - Add a new student
    const insertResult = await students.insertOne({
      name: "John Doe",
      age: 20,
      course: "Computer Science",
      email: "john@example.com"
    });
    console.log("Inserted student with id:", insertResult.insertedId);
    
    // FIND - Get all students
    const allStudents = await students.find({}).toArray();
    console.log("All students:", allStudents);
    
    // FIND - Get one student
    const john = await students.findOne({ name: "John Doe" });
    console.log("Found John:", john);
    
    // UPDATE - Change John's age
    const updateResult = await students.updateOne(
      { name: "John Doe" },
      { $set: { age: 21 } }
    );
    console.log("Updated:", updateResult.modifiedCount);
    
    // FIND again to see the change
    const updatedJohn = await students.findOne({ name: "John Doe" });
    console.log("Updated John:", updatedJohn);
    
    // DELETE - Remove John
    const deleteResult = await students.deleteOne({ name: "John Doe" });
    console.log("Deleted:", deleteResult.deletedCount);
    
  } catch (error) {
    console.error("Error:", error);
  } finally {
    // Close connection
    await client.close();
    console.log("Connection closed");
  }
}

main();
```

---

## Using Environment Variables for Connection

Never hardcode your database credentials

Create a .env file

```text
MONGODB_URI=mongodb+srv://admin:yourpassword@cluster0.abc123.mongodb.net/
DB_NAME=school
```

Use in your code

```javascript
require("dotenv").config();
const { MongoClient } = require("mongodb");

const url = process.env.MONGODB_URI;
const dbName = process.env.DB_NAME;

async function connect() {
  const client = new MongoClient(url);
  await client.connect();
  console.log("Connected to MongoDB");
  return client;
}
```

---

## Practice Exercises

### Exercise 1

Create a database named "library"

Create a collection named "books"

Insert 3 books with

```text
title
author
year
price
```

### Exercise 2

Find all books published after 2010

### Exercise 3

Update a book's price

### Exercise 4

Delete a book by its title

### Exercise 5

Find all books by a specific author

---

## Interview Questions

### What is MongoDB

MongoDB is a NoSQL database that stores data in JSON-like documents

### What is the difference between SQL and MongoDB

SQL uses tables and rows with fixed schema
MongoDB uses documents and collections with flexible schema

### What is a collection in MongoDB

A collection is a group of documents, similar to a table in SQL

### What is a document in MongoDB

A document is a single record, similar to a row in SQL

### What is _id in MongoDB

_id is a unique identifier automatically created for each document

### What is MongoDB Atlas

MongoDB Atlas is the cloud version of MongoDB hosted by MongoDB

### Do you need to create a database before using it

No
MongoDB creates the database when you first insert data

---

## Summary

In this session, you learned

* What MongoDB is
* Difference between SQL and NoSQL
* Important MongoDB terms
* How MongoDB stores data as documents
* How to set up MongoDB Atlas (cloud)
* How to connect to MongoDB from Node.js
* What MongoDB Compass is
* How to insert, find, update, delete documents
* Basic query operators like $gt and $lte

You now know how to use MongoDB with Node.js

---