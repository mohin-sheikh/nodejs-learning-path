## Table of Contents

* [Review of CRUD](#review-of-crud)
* [From Array to MongoDB](#from-array-to-mongodb)
* [Setting Up the Project](#setting-up-the-project)
* [Adding Sample Data with a Seed Script](#adding-sample-data-with-a-seed-script)
* [Connecting to MongoDB](#connecting-to-mongodb)
* [Validation and Response Format](#validation-and-response-format)
* [Checking the Id with Middleware](#checking-the-id-with-middleware)
* [Create Operation - Insert](#create-operation---insert)
* [Read Operation - Find](#read-operation---find)
* [Filtering with Query Strings](#filtering-with-query-strings)
* [Sorting and Limiting](#sorting-and-limiting)
* [Choosing Fields with Projection](#choosing-fields-with-projection)
* [Update Operation - Update](#update-operation---update)
* [Update Operators](#update-operators)
* [Delete Operation - Delete](#delete-operation---delete)
* [Unique Emails and 409 Conflict](#unique-emails-and-409-conflict)
* [Query Operators](#query-operators)
* [Error Handling](#error-handling)
* [Complete CRUD Example](#complete-crud-example)
* [Testing the API](#testing-the-api)
* [Beginner Mistakes](#beginner-mistakes)
* [Practice Exercises](#practice-exercises)
* [Interview Questions](#interview-questions)

---

## Review of CRUD

CRUD is the same whether using an array or a database

| Operation | HTTP Method | MongoDB Method                          |
| --------- | ----------- | --------------------------------------- |
| Create    | POST        | `insertOne()`, `insertMany()`           |
| Read      | GET         | `find()`, `findOne()`                   |
| Update    | PUT         | `findOneAndReplace()` (replace everything) |
| Update    | PATCH       | `updateOne()`, `findOneAndUpdate()` with `$set` |
| Delete    | DELETE      | `deleteOne()`, `deleteMany()`           |

The difference is that MongoDB saves data permanently

No data loss when server restarts

![An HTTP request becomes a MongoDB command and the result becomes JSON](images/18-mongodb-crud-operations/request-to-db.gif)

---

## From Array to MongoDB

In Session 15 we built a student API with an array. This session builds the **same API** with MongoDB. Same routes, same response format, same validation. Only the storage changes.

| Task           | Array (Session 15)                       | MongoDB (this session)                          |
| -------------- | ---------------------------------------- | ----------------------------------------------- |
| New id         | `nextId++`                               | MongoDB creates an ObjectId                     |
| Add            | `students.push(student)`                 | `await insertOne(student)`                      |
| Get all        | `students`                               | `await find({}).toArray()`                      |
| Get one        | `students.find((s) => s.id === id)`      | `await findOne({ _id: new ObjectId(id) })`      |
| Filter         | `students.filter(...)`                   | `find({ course: "Physics" })`                   |
| Remove         | `findIndex` + `splice`                   | `await deleteOne({ _id: ... })`                 |
| Id in the URL  | `Number(req.params.id)`                  | `new ObjectId(req.params.id)`                   |
| Duplicate email | `students.some(...)` before saving      | A unique index (MongoDB checks for you)         |

![Array code on the left, MongoDB code on the right](images/18-mongodb-crud-operations/array-to-mongo.gif)

Every MongoDB method talks to the database over the network, so every call needs `await` (Session 03), and the route must be `async`.

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

Create a .env file (Session 16)

```text
MONGODB_URI=mongodb+srv://yourusername:yourpassword@cluster0.abc123.mongodb.net/
DB_NAME=school
PORT=3000
```

For a local MongoDB, use `MONGODB_URI=mongodb://127.0.0.1:27017` (Session 17)

Create a .gitignore

```text
node_modules/
.env
```

Project structure

```text
mongodb-crud/
├── .env
├── .gitignore
├── package.json
├── db.js          connection code
├── seed.js        fills the database with sample students
├── server.js      Express routes
└── test-api.js    tests every route
```

Add scripts to package.json (Session 02)

```json
{
  "scripts": {
    "start": "node server.js",
    "dev": "node --watch server.js",
    "seed": "node seed.js"
  }
}
```

---

## Adding Sample Data with a Seed Script

An empty database is hard to test. A **seed script** fills it with sample data. You run it whenever you want a fresh start.

seed.js

```javascript
require("dotenv").config({ quiet: true });
const { MongoClient } = require("mongodb");

const students = [
  { name: "John Doe", age: 20, course: "Computer Science", email: "john@example.com" },
  { name: "Jane Smith", age: 22, course: "Mathematics", email: "jane@example.com" },
  { name: "Mike Johnson", age: 21, course: "Physics", email: "mike@example.com" },
  { name: "Sara Khan", age: 24, course: "Computer Science", email: "sara@example.com" },
  { name: "Tom Brown", age: 19, course: "Mathematics", email: "tom@example.com" }
];

async function seed() {
  const client = new MongoClient(process.env.MONGODB_URI);

  try {
    await client.connect();
    const collection = client.db(process.env.DB_NAME).collection("students");

    const deleted = await collection.deleteMany({});
    console.log("Removed old students:", deleted.deletedCount);

    const result = await collection.insertMany(
      students.map((s) => ({ ...s, createdAt: new Date() }))
    );
    console.log("Added students:", result.insertedCount);
  } catch (error) {
    console.error("Seeding failed:", error.message);
  } finally {
    await client.close();
  }
}

seed();
```

Run it

```bash
npm run seed
```

Output

```text
Removed old students: 0
Added students: 5
```

Run it again and it says `Removed old students: 5`. The seed always leaves exactly 5 students.

`students.map((s) => ({ ...s, createdAt: new Date() }))` makes a copy of each student with a `createdAt` date added. The `( )` around `{ }` tells JavaScript the arrow function returns an object.

Never run a seed script against a real production database. `deleteMany({})` removes every student.

---

## Connecting to MongoDB

Connect **once** when the server starts, then reuse the same connection in every route. Connecting takes time (often 100 ms or more to Atlas), so connecting inside each route would make every request slow.

![Connect once at startup, every request reuses the connection](images/18-mongodb-crud-operations/connect-once.gif)

Create a file named db.js

```javascript
const { MongoClient } = require("mongodb");

let db;

async function connectToDB() {
  const client = new MongoClient(process.env.MONGODB_URI);
  await client.connect();
  console.log("Connected to MongoDB");

  db = client.db(process.env.DB_NAME);

  // No two students may have the same email
  await db.collection("students").createIndex({ email: 1 }, { unique: true });
}

function getDB() {
  if (!db) {
    throw new Error("Database not connected. Call connectToDB first");
  }
  return db;
}

module.exports = { connectToDB, getDB };
```

| Code              | Meaning                                                    |
| ----------------- | ---------------------------------------------------------- |
| `let db;`         | Stores the database after connecting. Shared by every route |
| `connectToDB()`   | Connects once and saves the database in `db`               |
| `getDB()`         | Gives the saved database to any file that needs it         |
| `createIndex(...)` | Explained in [Unique Emails and 409 Conflict](#unique-emails-and-409-conflict) |

This works because a module runs only once and is cached (Session 04). Every file that requires db.js gets the same `db`.

`process.env` is read inside `connectToDB()`, not at the top of db.js. By the time the function runs, server.js has already loaded .env.

Now in server.js, connect first, then start listening

```javascript
require("dotenv").config({ quiet: true });
const express = require("express");
const { ObjectId } = require("mongodb");
const { connectToDB, getDB } = require("./db");

const app = express();
const PORT = process.env.PORT || 3000;

app.use(express.json());

// Short helper so every route can write students().find(...)
function students() {
  return getDB().collection("students");
}

// ... routes go here ...

// Connect first, then start the server
async function startServer() {
  try {
    await connectToDB();
  } catch (error) {
    console.error("Database connection failed:", error.message);
    process.exit(1);
  }

  app.listen(PORT, (err) => {
    if (err) {
      console.log("Could not start server:", err.message);
      return;
    }

    console.log(`Server running on port ${PORT}`);
  });
}

startServer();
```

Output

```text
Connected to MongoDB
Server running on port 3000
```

If the database is not reachable, the server does not start at all

```text
Database connection failed: connect ECONNREFUSED 127.0.0.1:27017
```

A server without its database cannot answer anything correctly, so stopping with `process.exit(1)` (Session 07) is better than running half-broken.

---

## Validation and Response Format

We reuse the response format from Session 15

| Field     | When                        |
| --------- | --------------------------- |
| `success` | Always, true or false       |
| `data`    | The student or the list     |
| `count`   | Number of items in a list   |
| `message` | Explains an error           |
| `errors`  | List of validation problems |

And the same validation function

```javascript
function validateStudent(student) {
  const errors = [];

  if (typeof student.name !== "string" || student.name.trim() === "") {
    errors.push("Name is required");
  }

  if (typeof student.age !== "number" || student.age < 18 || student.age > 60) {
    errors.push("Age must be a number between 18 and 60");
  }

  if (typeof student.email !== "string" || !student.email.includes("@")) {
    errors.push("A valid email is required");
  }

  return errors;
}
```

Validation matters even more now. MongoDB has no fixed structure (Session 17), so it saves whatever you give it. If you save `age: "abc"`, a later search like `{ age: { $gt: 18 } }` simply never finds that student.

---

## Checking the Id with Middleware

Four routes use `/students/:id`: GET, PUT, PATCH and DELETE all need a valid ObjectId. Instead of repeating the check in every route, write it once as middleware (Session 14)

```javascript
// Middleware: stop bad ids before the route runs
function checkId(req, res, next) {
  if (!ObjectId.isValid(req.params.id)) {
    return res.status(400).json({
      success: false,
      message: "Invalid student id"
    });
  }
  next();
}
```

Use it on a single route by putting it before the handler

```javascript
app.get("/students/:id", checkId, async (req, res) => { ... });
```

![checkId stops a bad id with 400, a good id continues to the route](images/18-mongodb-crud-operations/check-id.gif)

| id in the URL                | `ObjectId.isValid()` | Result                     |
| ---------------------------- | -------------------- | -------------------------- |
| `123`                        | false                | 400 Invalid student id     |
| `6ac203b2b7392100f72ccf0Z`   | false (Z is not hex) | 400 Invalid student id     |
| `6ac203b2b7392100f72ccf05`   | true                 | Route runs, then 200 or 404 |

Without this check, `new ObjectId("123")` throws a `BSONError` (Session 17) and the client gets a 500. A bad id is the client's mistake, so 400 is the correct answer.

Valid but not found is different: the id has the right shape, but no student has it. That is 404.

---

## Create Operation - Insert

POST /students - Add a new student

```javascript
app.post("/students", async (req, res) => {
  const { name, age, course, email } = req.body || {};

  const newStudent = {
    name,
    age,
    course: course || "Not specified",
    email,
    createdAt: new Date()
  };

  const errors = validateStudent(newStudent);
  if (errors.length > 0) {
    return res.status(400).json({ success: false, message: "Validation failed", errors });
  }

  await students().insertOne(newStudent);

  // insertOne added _id to newStudent
  res.status(201).json({ success: true, data: newStudent });
});
```

Response

```json
{
  "success": true,
  "data": {
    "name": "Sarah",
    "age": 23,
    "course": "Biology",
    "email": "sarah@example.com",
    "createdAt": "2026-10-04T07:59:26.964Z",
    "_id": "6ac2075ec79d0b2e6df5622a"
  }
}
```

![insertOne writes the student and adds _id to the same object](images/18-mongodb-crud-operations/insert-adds-id.gif)

Things to notice

* `insertOne()` adds the new `_id` to the object you pass in. That is why we can send `newStudent` back and it has an `_id`.
* `_id` is shown last because it was added last.
* We build `newStudent` ourselves from known fields. Saving `req.body` directly would let a client store any field, like `"role": "admin"`.
* `res.json()` turns the ObjectId and the Date into strings. The client receives `"_id": "6ac2..."` and an ISO date string.
* `req.body || {}` protects against a request without a JSON body (Session 12).

Insert multiple students at once

```javascript
app.post("/students/bulk", async (req, res) => {
  const list = (req.body || {}).students;

  if (!Array.isArray(list) || list.length === 0) {
    return res.status(400).json({ success: false, message: "Send { students: [ ... ] } with at least one student" });
  }

  const newStudents = [];

  for (let i = 0; i < list.length; i++) {
    const s = list[i];
    const student = { name: s.name, age: s.age, course: s.course || "Not specified", email: s.email, createdAt: new Date() };

    const errors = validateStudent(student);
    if (errors.length > 0) {
      return res.status(400).json({ success: false, message: `Student number ${i + 1} is not valid`, errors });
    }

    newStudents.push(student);
  }

  const result = await students().insertMany(newStudents);

  res.status(201).json({ success: true, count: result.insertedCount, data: newStudents });
});
```

`Array.isArray(list)` is true only for a real array. It rejects a string or an object, which also have a `length`.

Every student is checked **before** anything is saved, so one bad student stops the whole request with 400.

`insertMany()` stops at the first database error. If the 2nd of 3 students has an email that already exists, the 1st is saved, the 2nd and 3rd are not, and the client gets 409. We tested this.

---

## Read Operation - Find

GET /students/:id - Get one student

```javascript
app.get("/students/:id", checkId, async (req, res) => {
  const student = await students().findOne({ _id: new ObjectId(req.params.id) });

  if (!student) {
    return res.status(404).json({ success: false, message: "Student not found" });
  }

  res.status(200).json({ success: true, data: student });
});
```

`findOne()` returns `null` when nothing matches, so `!student` means 404.

GET /students - Get all students

```javascript
app.get("/students", async (req, res) => {
  const list = await students().find({}).toArray();

  res.status(200).json({ success: true, count: list.length, data: list });
});
```

The next two sections add filtering and sorting to this route.

---

## Filtering with Query Strings

In Session 13 you filtered an array with `?course=` and `?minAge=`. With MongoDB you build a **filter object** from the query string and let the database do the work.

![The query string becomes a MongoDB filter object](images/18-mongodb-crud-operations/query-to-filter.gif)

```javascript
app.get("/students", async (req, res) => {
  const filter = {};

  if (req.query.course) {
    filter.course = req.query.course;
  }

  if (req.query.minAge || req.query.maxAge) {
    filter.age = {};

    if (req.query.minAge) {
      const minAge = Number(req.query.minAge);
      if (Number.isNaN(minAge)) {
        return res.status(400).json({ success: false, message: "minAge must be a number" });
      }
      filter.age.$gte = minAge;
    }

    if (req.query.maxAge) {
      const maxAge = Number(req.query.maxAge);
      if (Number.isNaN(maxAge)) {
        return res.status(400).json({ success: false, message: "maxAge must be a number" });
      }
      filter.age.$lte = maxAge;
    }
  }

  const list = await students().find(filter).toArray();

  res.status(200).json({ success: true, count: list.length, data: list });
});
```

| URL                                    | filter object                           | Students found (seed data)        |
| -------------------------------------- | --------------------------------------- | --------------------------------- |
| `/students`                            | `{}`                                    | All 5                             |
| `/students?course=Mathematics`         | `{ course: "Mathematics" }`             | Jane Smith, Tom Brown             |
| `/students?minAge=20&maxAge=22`        | `{ age: { $gte: 20, $lte: 22 } }`       | John Doe, Jane Smith, Mike Johnson |
| `/students?minAge=abc`                 | not built                               | 400 minAge must be a number       |

`Number()` is required. `req.query.minAge` is the string `"20"`, and MongoDB does not match the string `"20"` against the number `20` (Session 17).

Note: `?course=mathematics` (small m) finds nothing, because MongoDB comparisons are case-sensitive. Session 20 shows a case-insensitive search with `$regex`.

---

## Sorting and Limiting

`find()` returns a cursor (Session 17). Before `toArray()`, you can tell the cursor how to sort and how many documents to return.

| Method         | Example                  | Meaning                         |
| -------------- | ------------------------ | ------------------------------- |
| `.sort()`      | `.sort({ age: 1 })`      | Smallest age first (1 = up)     |
| `.sort()`      | `.sort({ age: -1 })`     | Biggest age first (-1 = down)   |
| `.limit()`     | `.limit(3)`              | At most 3 documents             |
| `.skip()`      | `.skip(2)`               | Skip the first 2                |
| `countDocuments()` | `countDocuments({ course: "Computer Science" })` | How many match (here 2) |

```javascript
// The 3 oldest students
const oldest = await students().find({}).sort({ age: -1 }).limit(3).toArray();

// Students in alphabetical order
const byName = await students().find({}).sort({ name: 1 }).toArray();
```

Results with the seed data

```text
oldest: Sara Khan, Jane Smith, Mike Johnson
byName: Jane Smith, John Doe, Mike Johnson, Sara Khan, Tom Brown
```

![sort orders the documents, skip jumps over some, limit cuts the list](images/18-mongodb-crud-operations/sort-skip-limit.gif)

Without `.sort()`, MongoDB returns documents in no guaranteed order. Usually it looks like insert order, but never rely on that.

`skip()` and `limit()` together give pages: page 2 with 10 per page is `.skip(10).limit(10)`. Session 20 builds full pagination with them.

Add `?sort=age` and `?sort=-age` to GET /students

```javascript
  // ?sort=age (small to big) or ?sort=-age (big to small)
  const sort = {};
  if (req.query.sort) {
    const field = req.query.sort.replace("-", "");
    if (!["name", "age", "createdAt"].includes(field)) {
      return res.status(400).json({ success: false, message: "You can sort by name, age or createdAt" });
    }
    sort[field] = req.query.sort.startsWith("-") ? -1 : 1;
  }

  const list = await students().find(filter).sort(sort).toArray();
```

| URL                     | sort object      |
| ----------------------- | ---------------- |
| `/students`             | `{}` (no order)  |
| `/students?sort=age`    | `{ age: 1 }`     |
| `/students?sort=-age`   | `{ age: -1 }`    |
| `/students?sort=password` | 400            |

`sort[field] = ...` uses bracket notation because the field name is in a variable (Session 16 used the same idea with `process.env[name]`). The allowed list stops clients from sorting by fields you do not want them to see.

---

## Choosing Fields with Projection

Sometimes you do not want to send every field. A **projection** picks which fields come back.

```javascript
// Only name and age (_id is included unless you remove it)
const list = await students().find(
  { course: "Physics" },
  { projection: { name: 1, age: 1 } }
).toArray();
```

```text
[
  {
    _id: new ObjectId('6ac2075ea6f5677188a04a5c'),
    name: 'Mike Johnson',
    age: 21
  }
]
```

```javascript
// Everything except _id and email
const list = await students().find(
  { course: "Physics" },
  { projection: { _id: 0, email: 0 } }
).toArray();
```

```text
[
  {
    name: 'Mike Johnson',
    age: 21,
    course: 'Physics',
    createdAt: 2026-10-04T07:59:26.845Z
  }
]
```

| Value | Meaning      |
| ----- | ------------ |
| `1`   | Include this field |
| `0`   | Leave this field out |

Do not mix 1 and 0 in one projection (except for `_id`). Either list what you want, or list what you do not want.

This is how you will hide passwords in Session 23: a user document has a password, but the API never sends it.

---

## Update Operation - Update

In Session 15 you learned the difference

| Method | Body needs        | MongoDB method           |
| ------ | ----------------- | ------------------------ |
| PUT    | Every field       | `findOneAndReplace()`    |
| PATCH  | Only what changes | `findOneAndUpdate()` with `$set` |

PUT /students/:id - Replace a student

```javascript
app.put("/students/:id", checkId, async (req, res) => {
  const { name, age, course, email } = req.body || {};

  const replacement = {
    name,
    age,
    course: course || "Not specified",
    email,
    updatedAt: new Date()
  };

  const errors = validateStudent(replacement);
  if (errors.length > 0) {
    return res.status(400).json({ success: false, message: "Validation failed", errors });
  }

  const updated = await students().findOneAndReplace(
    { _id: new ObjectId(req.params.id) },
    replacement,
    { returnDocument: "after" }
  );

  if (!updated) {
    return res.status(404).json({ success: false, message: "Student not found" });
  }

  res.status(200).json({ success: true, data: updated });
});
```

`findOneAndReplace()` swaps the whole document for the new one. The `_id` stays the same. A replacement must not contain `$set`, otherwise the driver throws `Replacement document must not contain atomic operators`.

PATCH /students/:id - Change some fields

```javascript
app.patch("/students/:id", checkId, async (req, res) => {
  const body = req.body || {};
  const id = new ObjectId(req.params.id);

  const current = await students().findOne({ _id: id });
  if (!current) {
    return res.status(404).json({ success: false, message: "Student not found" });
  }

  const changes = {};
  for (const key of ["name", "age", "course", "email"]) {
    if (body[key] !== undefined) {
      changes[key] = body[key];
    }
  }

  // Check the student as it will look after the change
  const errors = validateStudent({ ...current, ...changes });
  if (errors.length > 0) {
    return res.status(400).json({ success: false, message: "Validation failed", errors });
  }

  changes.updatedAt = new Date();

  const updated = await students().findOneAndUpdate(
    { _id: id },
    { $set: changes },
    { returnDocument: "after" }
  );

  res.status(200).json({ success: true, data: updated });
});
```

Send a PATCH with `{ "age": 24 }`

```json
{
  "success": true,
  "data": {
    "_id": "6ac2075ec79d0b2e6df5622a",
    "name": "Sarah",
    "age": 24,
    "course": "Biology",
    "email": "sarah@example.com",
    "createdAt": "2026-10-04T07:59:26.964Z",
    "updatedAt": "2026-10-04T07:59:27.000Z"
  }
}
```

![PUT swaps the whole document, PATCH changes only the sent fields](images/18-mongodb-crud-operations/put-vs-patch.gif)

| Code                                    | Meaning                                                    |
| --------------------------------------- | ---------------------------------------------------------- |
| `findOne` first                         | We need the current student to validate the final result, and to send 404 |
| `["name", "age", "course", "email"]`    | Only these fields may change. `_id` or `role` in the body are ignored |
| `body[key] !== undefined`               | Only fields the client actually sent                       |
| `{ ...current, ...changes }`            | The student as it will look after the change               |
| `returnDocument: "after"`               | Return the student after the update. The default `"before"` returns the old version |

The `!== undefined` check matters. `{ $set: { course: undefined } }` does not skip the field. We tested it: MongoDB saves `course: null`, and the course is lost.

`findOneAndUpdate()` returns the updated document, or `null` if nothing matched. With `updateOne()` you only get counts (`matchedCount`, `modifiedCount`, Session 17) and need a second `findOne()` to get the student.

Update multiple students

```javascript
const result = await students().updateMany(
  { course: "Mathematics" },
  { $set: { course: "Maths" } }
);

console.log(result.matchedCount, result.modifiedCount);
```

Output

```text
2 2
```

---

## Update Operators

`$set` is not the only update operator

| Operator  | Example                                   | Result                          |
| --------- | ----------------------------------------- | ------------------------------- |
| `$set`    | `{ $set: { age: 21 } }`                   | Set the field to a value        |
| `$unset`  | `{ $unset: { course: "" } }`              | Remove the field completely     |
| `$inc`    | `{ $inc: { age: 1 } }`                    | Add 1 (use -1 to subtract)      |
| `$push`   | `{ $push: { skills: "Node.js" } }`        | Add an item to an array         |
| `$pull`   | `{ $pull: { skills: "Node.js" } }`        | Remove an item from an array    |

```javascript
await students().updateOne({ name: "John Doe" }, { $inc: { age: 1 } });
await students().updateOne({ name: "John Doe" }, { $push: { skills: "Node.js" } });
await students().updateOne({ name: "John Doe" }, { $push: { skills: "MongoDB" } });
await students().updateOne({ name: "John Doe" }, { $pull: { skills: "Node.js" } });
await students().updateOne({ name: "John Doe" }, { $unset: { course: "" } });

console.log(await students().findOne({ name: "John Doe" }, { projection: { _id: 0 } }));
```

Output

```text
{
  name: 'John Doe',
  age: 21,
  email: 'john@example.com',
  createdAt: 2026-10-04T07:59:26.845Z,
  skills: [ 'MongoDB' ]
}
```

![$inc, $push, $pull and $unset change the document step by step](images/18-mongodb-crud-operations/update-operators.gif)

Why `$inc` instead of reading the age, adding 1 and saving it? If two requests do "read, add, save" at the same moment, both read 20 and both save 21. One increase is lost. `$inc` happens inside the database in one step, so both increases count. This matters for things like product stock or like counters.

`$push` creates the array if it does not exist yet. The value given to `$unset` (`""`) does not matter, only the field name.

---

## Delete Operation - Delete

DELETE /students/:id - Delete a student

```javascript
app.delete("/students/:id", checkId, async (req, res) => {
  const result = await students().deleteOne({ _id: new ObjectId(req.params.id) });

  if (result.deletedCount === 0) {
    return res.status(404).json({ success: false, message: "Student not found" });
  }

  res.status(200).json({ success: true, message: "Student deleted successfully" });
});
```

`deletedCount === 0` means nothing matched, so 404.

If you want to send the deleted student back, use `findOneAndDelete()`. It returns the document that was removed, or `null`.

Delete multiple students

```javascript
const result = await students().deleteMany({ course: "Physics" });
console.log(`${result.deletedCount} students deleted`);
```

Be very careful with `deleteMany` in a route. If the filter is built from the request and ends up empty (`{}`), every student is deleted. Most real APIs do not offer "delete many" to normal users at all.

---

## Unique Emails and 409 Conflict

In Session 15 you checked for a duplicate email with `students.some(...)` before saving. With a database, a better way is a **unique index**: MongoDB itself refuses a second document with the same email.

We created it in db.js

```javascript
await db.collection("students").createIndex({ email: 1 }, { unique: true });
```

| Part              | Meaning                                                |
| ----------------- | ------------------------------------------------------ |
| `{ email: 1 }`    | Build an index on the email field (1 = ascending order) |
| `{ unique: true }` | No two documents may have the same email              |

An **index** is like the index at the back of a book. Instead of reading every page to find "MongoDB", you look it up in the index. MongoDB uses it to find emails fast, and to check for duplicates.

`createIndex()` is safe to run on every startup. If the index already exists, nothing changes. If the collection already has duplicate emails, creating the index fails, so clean the data first.

Now a duplicate insert throws

```text
MongoServerError: E11000 duplicate key error collection: school.students index: email_1 dup key: { email: "sarah@example.com" }
```

![The unique index blocks the second sarah@example.com and the API answers 409](images/18-mongodb-crud-operations/unique-email.gif)

We turn it into a clean 409 in the error handler (next section). The error has `err.code === 11000`.

Why not check with `findOne({ email })` first? Two requests arriving at the same moment could both see "no such email" and both insert. The unique index cannot be fooled this way, because the database checks at the moment of writing.

The same index protects PATCH and PUT: changing a student's email to one that another student uses also gives 409.

---

## Query Operators

MongoDB provides operators for complex queries

Comparison operators

| Operator | Meaning               | Example                                   |
| -------- | --------------------- | ----------------------------------------- |
| `$eq`    | Equal                 | `{ age: { $eq: 20 } }` (same as `{ age: 20 }`) |
| `$ne`    | Not equal             | `{ course: { $ne: "Physics" } }`          |
| `$gt`    | Greater than          | `{ age: { $gt: 20 } }`                    |
| `$gte`   | Greater than or equal | `{ age: { $gte: 20 } }`                   |
| `$lt`    | Less than             | `{ age: { $lt: 20 } }`                    |
| `$lte`   | Less than or equal    | `{ age: { $lte: 20 } }`                   |
| `$in`    | Equal to any in list  | `{ course: { $in: ["Physics", "Mathematics"] } }` |
| `$nin`   | Not in list           | `{ course: { $nin: ["Physics", "Mathematics"] } }` |

Logical operators

| Operator | Meaning              | Example                                          |
| -------- | -------------------- | ------------------------------------------------ |
| `$and`   | All conditions true  | `{ $and: [{ age: 20 }, { course: "Computer Science" }] }` |
| `$or`    | Any condition true   | `{ $or: [{ age: { $lt: 20 } }, { course: "Physics" }] }` |
| `$nor`   | No condition true    | `{ $nor: [{ age: 20 }, { course: "Physics" }] }` |
| `$not`   | Opposite of a condition | `{ age: { $not: { $gt: 21 } } }`              |

Other useful operators

| Operator  | Meaning                     | Example                               |
| --------- | --------------------------- | ------------------------------------- |
| `$exists` | Field is there (or not)     | `{ phone: { $exists: true } }`        |
| `$regex`  | Text matches a pattern      | `{ name: { $regex: "jo", $options: "i" } }` |

![Each operator lets through only the matching students](images/18-mongodb-crud-operations/operators-filter.gif)

Results with the seed data (all tested)

| Filter                                                   | Students found                       |
| -------------------------------------------------------- | ------------------------------------ |
| `{ age: { $gte: 20, $lte: 22 } }`                        | John Doe, Jane Smith, Mike Johnson   |
| `{ course: { $in: ["Physics", "Mathematics"] } }`        | Jane Smith, Mike Johnson, Tom Brown  |
| `{ course: { $ne: "Computer Science" } }`                | Jane Smith, Mike Johnson, Tom Brown  |
| `{ $or: [{ age: { $lt: 20 } }, { course: "Physics" }] }` | Mike Johnson, Tom Brown              |
| `{ name: { $regex: "jo", $options: "i" } }`              | John Doe, Mike Johnson               |
| `{ name: { $regex: "^J" } }`                             | John Doe, Jane Smith                 |

Things to know

* Several fields in one object already mean AND: `{ age: 20, course: "Computer Science" }` is the same as the `$and` example. Use `$and` only when you need two conditions on the **same** field in different objects.
* `$regex` with `$options: "i"` ignores capital letters. `"jo"` matches anywhere in the name, so "Mike **Jo**hnson" matches too. `^` means "starts with".
* In a regex, characters like `(`, `*`, `+` and `.` have special meanings. A search for `(` gives `MongoServerError: Regular expression is invalid: missing closing parenthesis`. Session 20 uses `$regex` for search.
* `$not` and `$ne` also match documents that do not have the field at all. A student without `age` matches `{ age: { $not: { $gt: 21 } } }`.

Example route with an operator

```javascript
// GET /students/older-than/21
app.get("/students/older-than/:age", async (req, res) => {
  const age = Number(req.params.age);

  if (Number.isNaN(age)) {
    return res.status(400).json({ success: false, message: "Age must be a number" });
  }

  const list = await students().find({ age: { $gt: age } }).toArray();

  res.status(200).json({ success: true, count: list.length, data: list });
});
```

---

## Error Handling

In Express 5, an error thrown in an `async` route goes to the error handler automatically (Session 15), so the routes above have no try / catch. One error handler at the end handles everything

```javascript
// 404 for unknown routes (Express 5: no "*")
app.use((req, res) => {
  res.status(404).json({ success: false, message: `Route ${req.method} ${req.url} not found` });
});

// Error handler (Session 14)
app.use((err, req, res, next) => {
  // The body was not valid JSON
  if (err.type === "entity.parse.failed") {
    return res.status(400).json({ success: false, message: "Request body is not valid JSON" });
  }

  // E11000: the unique email index blocked a duplicate
  if (err.code === 11000) {
    return res.status(409).json({ success: false, message: "Email already exists" });
  }

  console.error(err);
  res.status(500).json({ success: false, message: "Something went wrong on the server" });
});
```

![Each kind of error gets its own status code](images/18-mongodb-crud-operations/error-handler.gif)

| Error                              | Status | Why                                     |
| ---------------------------------- | ------ | --------------------------------------- |
| Body is broken JSON                | 400    | `express.json()` could not read it      |
| Duplicate email                    | 409    | Unique index, `err.code === 11000`      |
| Anything else (database down, bug) | 500    | Our problem, not the client's           |

Without the `entity.parse.failed` check, broken JSON would become a 500, because once you write your own error handler, Express no longer answers 400 for you.

Never send `err.message` from a database error to the client. It can contain collection names, index names and data. Log it with `console.error` instead.

---

## Complete CRUD Example

db.js is shown in [Connecting to MongoDB](#connecting-to-mongodb). Here is the complete server.js

```javascript
require("dotenv").config({ quiet: true });
const express = require("express");
const { ObjectId } = require("mongodb");
const { connectToDB, getDB } = require("./db");

const app = express();
const PORT = process.env.PORT || 3000;

app.use(express.json());

// Short helper so every route can write students().find(...)
function students() {
  return getDB().collection("students");
}

// Same validation as Session 15
function validateStudent(student) {
  const errors = [];

  if (typeof student.name !== "string" || student.name.trim() === "") {
    errors.push("Name is required");
  }

  if (typeof student.age !== "number" || student.age < 18 || student.age > 60) {
    errors.push("Age must be a number between 18 and 60");
  }

  if (typeof student.email !== "string" || !student.email.includes("@")) {
    errors.push("A valid email is required");
  }

  return errors;
}

// Middleware: stop bad ids before the route runs
function checkId(req, res, next) {
  if (!ObjectId.isValid(req.params.id)) {
    return res.status(400).json({
      success: false,
      message: "Invalid student id"
    });
  }
  next();
}

// CREATE - POST /students
app.post("/students", async (req, res) => {
  const { name, age, course, email } = req.body || {};

  const newStudent = {
    name,
    age,
    course: course || "Not specified",
    email,
    createdAt: new Date()
  };

  const errors = validateStudent(newStudent);
  if (errors.length > 0) {
    return res.status(400).json({ success: false, message: "Validation failed", errors });
  }

  await students().insertOne(newStudent);

  // insertOne added _id to newStudent
  res.status(201).json({ success: true, data: newStudent });
});

// CREATE MANY - POST /students/bulk
app.post("/students/bulk", async (req, res) => {
  const list = (req.body || {}).students;

  if (!Array.isArray(list) || list.length === 0) {
    return res.status(400).json({ success: false, message: "Send { students: [ ... ] } with at least one student" });
  }

  const newStudents = [];

  for (let i = 0; i < list.length; i++) {
    const s = list[i];
    const student = { name: s.name, age: s.age, course: s.course || "Not specified", email: s.email, createdAt: new Date() };

    const errors = validateStudent(student);
    if (errors.length > 0) {
      return res.status(400).json({ success: false, message: `Student number ${i + 1} is not valid`, errors });
    }

    newStudents.push(student);
  }

  const result = await students().insertMany(newStudents);

  res.status(201).json({ success: true, count: result.insertedCount, data: newStudents });
});

// READ ALL - GET /students?course=...&minAge=...&maxAge=...&sort=...
app.get("/students", async (req, res) => {
  const filter = {};

  if (req.query.course) {
    filter.course = req.query.course;
  }

  if (req.query.minAge || req.query.maxAge) {
    filter.age = {};

    if (req.query.minAge) {
      const minAge = Number(req.query.minAge);
      if (Number.isNaN(minAge)) {
        return res.status(400).json({ success: false, message: "minAge must be a number" });
      }
      filter.age.$gte = minAge;
    }

    if (req.query.maxAge) {
      const maxAge = Number(req.query.maxAge);
      if (Number.isNaN(maxAge)) {
        return res.status(400).json({ success: false, message: "maxAge must be a number" });
      }
      filter.age.$lte = maxAge;
    }
  }

  // ?sort=age (small to big) or ?sort=-age (big to small)
  const sort = {};
  if (req.query.sort) {
    const field = req.query.sort.replace("-", "");
    if (!["name", "age", "createdAt"].includes(field)) {
      return res.status(400).json({ success: false, message: "You can sort by name, age or createdAt" });
    }
    sort[field] = req.query.sort.startsWith("-") ? -1 : 1;
  }

  const list = await students().find(filter).sort(sort).toArray();

  res.status(200).json({ success: true, count: list.length, data: list });
});

// READ ONE - GET /students/:id
app.get("/students/:id", checkId, async (req, res) => {
  const student = await students().findOne({ _id: new ObjectId(req.params.id) });

  if (!student) {
    return res.status(404).json({ success: false, message: "Student not found" });
  }

  res.status(200).json({ success: true, data: student });
});

// REPLACE - PUT /students/:id (every field is required)
app.put("/students/:id", checkId, async (req, res) => {
  const { name, age, course, email } = req.body || {};

  const replacement = {
    name,
    age,
    course: course || "Not specified",
    email,
    updatedAt: new Date()
  };

  const errors = validateStudent(replacement);
  if (errors.length > 0) {
    return res.status(400).json({ success: false, message: "Validation failed", errors });
  }

  const updated = await students().findOneAndReplace(
    { _id: new ObjectId(req.params.id) },
    replacement,
    { returnDocument: "after" }
  );

  if (!updated) {
    return res.status(404).json({ success: false, message: "Student not found" });
  }

  res.status(200).json({ success: true, data: updated });
});

// UPDATE SOME FIELDS - PATCH /students/:id
app.patch("/students/:id", checkId, async (req, res) => {
  const body = req.body || {};
  const id = new ObjectId(req.params.id);

  const current = await students().findOne({ _id: id });
  if (!current) {
    return res.status(404).json({ success: false, message: "Student not found" });
  }

  const changes = {};
  for (const key of ["name", "age", "course", "email"]) {
    if (body[key] !== undefined) {
      changes[key] = body[key];
    }
  }

  // Check the student as it will look after the change
  const errors = validateStudent({ ...current, ...changes });
  if (errors.length > 0) {
    return res.status(400).json({ success: false, message: "Validation failed", errors });
  }

  changes.updatedAt = new Date();

  const updated = await students().findOneAndUpdate(
    { _id: id },
    { $set: changes },
    { returnDocument: "after" }
  );

  res.status(200).json({ success: true, data: updated });
});

// DELETE - DELETE /students/:id
app.delete("/students/:id", checkId, async (req, res) => {
  const result = await students().deleteOne({ _id: new ObjectId(req.params.id) });

  if (result.deletedCount === 0) {
    return res.status(404).json({ success: false, message: "Student not found" });
  }

  res.status(200).json({ success: true, message: "Student deleted successfully" });
});

// 404 for unknown routes (Express 5: no "*")
app.use((req, res) => {
  res.status(404).json({ success: false, message: `Route ${req.method} ${req.url} not found` });
});

// Error handler (Session 14)
app.use((err, req, res, next) => {
  // The body was not valid JSON
  if (err.type === "entity.parse.failed") {
    return res.status(400).json({ success: false, message: "Request body is not valid JSON" });
  }

  // E11000: the unique email index blocked a duplicate
  if (err.code === 11000) {
    return res.status(409).json({ success: false, message: "Email already exists" });
  }

  console.error(err);
  res.status(500).json({ success: false, message: "Something went wrong on the server" });
});

// Connect first, then start the server
async function startServer() {
  try {
    await connectToDB();
  } catch (error) {
    console.error("Database connection failed:", error.message);
    process.exit(1);
  }

  app.listen(PORT, (err) => {
    if (err) {
      console.log("Could not start server:", err.message);
      return;
    }

    console.log(`Server running on port ${PORT}`);
  });
}

startServer();
```

Route order matters (Session 13): `POST /students/bulk` is a different path from `POST /students`, and `GET /students/:id` only matches one path part after `/students/`, so the routes do not clash.

---

## Testing the API

Start the server

```bash
npm run seed
npm run dev
```

GET requests work in the browser

```text
http://localhost:3000/students
http://localhost:3000/students?course=Mathematics&sort=-age
http://localhost:3000/students?minAge=20&maxAge=22
```

For POST, PUT, PATCH and DELETE, use Postman or a test script with fetch, like in Session 15. The id is now an ObjectId, so the script takes it from the POST response

test-api.js

```javascript
const BASE = "http://localhost:3000/students";

async function send(method, url, body) {
  const res = await fetch(url, {
    method,
    headers: { "Content-Type": "application/json" },
    body: body ? JSON.stringify(body) : undefined
  });
  const data = await res.json();
  console.log(method, url.replace(BASE, "/students"), res.status, JSON.stringify(data));
  return data;
}

async function test() {
  const created = await send("POST", BASE, { name: "Sarah", age: 23, course: "Biology", email: "sarah@example.com" });
  const id = created.data._id;

  await send("POST", BASE, { name: "", age: "abc" });
  await send("POST", BASE, { name: "Copy", age: 30, email: "sarah@example.com" });
  await send("GET", BASE + "/" + id);
  await send("PATCH", BASE + "/" + id, { age: 24 });
  await send("PUT", BASE + "/" + id, { name: "Only name" });
  await send("GET", BASE + "/123");
  await send("DELETE", BASE + "/" + id);
  await send("GET", BASE + "/" + id);
}

test();
```

Run it in a second terminal

```bash
node test-api.js
```

Output (your ids and dates will be different)

```text
POST /students 201 {"success":true,"data":{"name":"Sarah","age":23,"course":"Biology","email":"sarah@example.com","createdAt":"2026-10-04T07:59:26.964Z","_id":"6ac2075ec79d0b2e6df5622a"}}
POST /students 400 {"success":false,"message":"Validation failed","errors":["Name is required","Age must be a number between 18 and 60","A valid email is required"]}
POST /students 409 {"success":false,"message":"Email already exists"}
GET /students/6ac2075ec79d0b2e6df5622a 200 {"success":true,"data":{"_id":"6ac2075ec79d0b2e6df5622a","name":"Sarah","age":23,"course":"Biology","email":"sarah@example.com","createdAt":"2026-10-04T07:59:26.964Z"}}
PATCH /students/6ac2075ec79d0b2e6df5622a 200 {"success":true,"data":{"_id":"6ac2075ec79d0b2e6df5622a","name":"Sarah","age":24,"course":"Biology","email":"sarah@example.com","createdAt":"2026-10-04T07:59:26.964Z","updatedAt":"2026-10-04T07:59:27.000Z"}}
PUT /students/6ac2075ec79d0b2e6df5622a 400 {"success":false,"message":"Validation failed","errors":["Age must be a number between 18 and 60","A valid email is required"]}
GET /students/123 400 {"success":false,"message":"Invalid student id"}
DELETE /students/6ac2075ec79d0b2e6df5622a 200 {"success":true,"message":"Student deleted successfully"}
GET /students/6ac2075ec79d0b2e6df5622a 404 {"success":false,"message":"Student not found"}
```

![The test script walks through every status code](images/18-mongodb-crud-operations/test-run.gif)

Every status code from Session 15 shows up: 201, 400, 409, 200, 404. Open Compass while testing to watch Sarah appear, change and disappear.

curl on macOS, Linux or Git Bash (replace the id with a real one)

```bash
curl -X POST http://localhost:3000/students -H "Content-Type: application/json" -d '{"name":"Sarah","age":23,"course":"Biology","email":"sarah@example.com"}'

curl -X PATCH http://localhost:3000/students/6ac2075ec79d0b2e6df5622a -H "Content-Type: application/json" -d '{"age":24}'

curl -X DELETE http://localhost:3000/students/6ac2075ec79d0b2e6df5622a
```

On Windows, use the Command Prompt or PowerShell commands from [Session 11](11-crud-operations-dummy-data.md#testing-your-api).

---

## Beginner Mistakes

### Mistake 1

Connecting to MongoDB inside every route.

Each request opens a new connection, which is slow and can reach the connection limit of a free Atlas cluster. Connect once in `startServer()`.

---

### Mistake 2

Starting the server before the database is connected.

```javascript
connectToDB();
app.listen(3000);
```

Without `await`, the first requests can arrive before `db` is set and fail with "Database not connected". Await the connection, then call `app.listen()`.

---

### Mistake 3

Searching with `req.params.id` directly.

`findOne({ _id: req.params.id })` always returns `null`. Use `new ObjectId(req.params.id)`, after checking it with `ObjectId.isValid()`.

---

### Mistake 4

Forgetting `Number()` for query values.

`find({ age: { $gte: req.query.minAge } })` compares with the string `"20"` and finds nothing.

---

### Mistake 5

Saving `req.body` directly.

```javascript
await students().insertOne(req.body);
```

The client decides which fields are saved, including `role: "admin"` or a 10 MB string. Build the object from known fields and validate it.

---

### Mistake 6

Putting `undefined` in `$set`.

`{ $set: { course: req.body.course } }` saves `course: null` when the client did not send a course. Only add fields that are not `undefined`.

---

### Mistake 7

Checking `modifiedCount` for 404.

Sending the same PATCH twice gives `modifiedCount: 0` the second time, although the student exists. Use `matchedCount`, or `findOneAndUpdate()` and check for `null`.

---

### Mistake 8

Checking duplicates only with `findOne()`.

Two requests at the same moment can both pass the check. Use a unique index and handle `err.code === 11000`.

---

### Mistake 9

Forgetting `returnDocument: "after"`.

`findOneAndUpdate()` returns the **old** document by default, so the client sees the values from before the change.

---

## Practice Exercises

### Exercise 1

Create a products API with MongoDB

Each product should have

* name
* price
* category
* stock

Implement all CRUD operations, with a `validateProduct` function, the `checkId` middleware and the same response format

### Exercise 2

Write a seed.js that adds 8 products in 3 categories

### Exercise 3

Add filters to GET /products

```text
GET /products?category=books
GET /products?minPrice=100&maxPrice=500
GET /products?sort=-price
```

Return 400 if minPrice or maxPrice is not a number

### Exercise 4

Add a route to change the stock

PATCH /products/:id/stock

Send `{ "change": -2 }` in the body (2 items were sold). Use `$inc`. Return 400 if change is not a number

### Exercise 5

Add a unique index on the product name. Creating a product with an existing name should return 409

### Exercise 6

Add GET /products/:id?fields=name,price that returns only those fields, using a projection

Hint: `"name,price".split(",")` gives `["name", "price"]`

### Exercise 7

Add a `tags` array to products. Write PATCH /products/:id/tags that adds a tag with `$push`, and DELETE /products/:id/tags/:tag that removes one with `$pull`

### Exercise 8

Write a test-api.js for your products API that shows 201, 400, 404 and 409

---

## Interview Questions

### What is the difference between insertOne and insertMany

insertOne inserts a single document
insertMany inserts multiple documents at once. It stops at the first error, and documents before the error stay saved

### What is ObjectId in MongoDB

ObjectId is the default type of _id. A 24 character hex value that MongoDB creates for each document

### Why do we need to check ObjectId.isValid

new ObjectId() throws an error for a badly formed id. Checking first lets us answer 400 instead of crashing with 500

### What is the difference between 400 and 404 for an id

400: the id is not a valid ObjectId. 404: the id is valid but no document has it

### Why connect to the database only once

Connecting is slow. One shared connection is reused by every request. The driver keeps a pool of connections ready

### What does the $set operator do

$set updates only the fields you specify without affecting other fields

### What is the difference between updateOne and findOneAndUpdate

updateOne returns counts (matchedCount, modifiedCount). findOneAndUpdate returns the document itself, before or after the change (returnDocument)

### What is the difference between updateOne and updateMany

updateOne updates the first matching document
updateMany updates all matching documents

### How do you implement PUT and PATCH with MongoDB

PUT replaces the whole document (findOneAndReplace). PATCH changes only the sent fields with $set (findOneAndUpdate or updateOne)

### What does $inc do and why use it

It adds a number to a field inside the database in one step. Two requests at the same time cannot overwrite each other

### What is a unique index

An index that does not allow two documents with the same value in a field. MongoDB throws error code 11000 (E11000) on a duplicate

### How do you return 409 for a duplicate email

Create a unique index on email and, in the error handler, check err.code === 11000

### What is a projection

The second argument of find that chooses which fields are returned, for example { projection: { password: 0 } }

### How do you sort and limit results

find().sort({ age: -1 }).limit(3). 1 means ascending, -1 descending

### What does the $gt operator mean

$gt means greater than

### How do you find documents with age between 20 and 30

{ age: { $gte: 20, $lte: 30 } }

### What is the difference between $in and $or

$in checks one field against a list of values. $or combines different conditions, possibly on different fields

---
