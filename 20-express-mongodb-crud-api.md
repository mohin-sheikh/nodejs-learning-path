## Table of Contents

* [Project Overview](#project-overview)
* [Project Structure](#project-structure)
* [How a Request Travels Through the Folders](#how-a-request-travels-through-the-folders)
* [Setting Up the Project](#setting-up-the-project)
* [Creating the Student Model](#creating-the-student-model)
* [Creating Controllers](#creating-controllers)
* [Searching, Filtering and Sorting](#searching-filtering-and-sorting)
* [Pagination](#pagination)
* [Creating Routes](#creating-routes)
* [The Error Handler Middleware](#the-error-handler-middleware)
* [Putting It All Together](#putting-it-all-together)
* [Adding Sample Data](#adding-sample-data)
* [Testing the API](#testing-the-api)
* [Beginner Mistakes](#beginner-mistakes)
* [Practice Exercises](#practice-exercises)
* [Interview Questions](#interview-questions)

---

## Project Overview

We will build a complete Student Management API

This API will have

| Method | URL                         | What it does                                   |
| ------ | --------------------------- | ---------------------------------------------- |
| GET    | `/api/students`             | List students, with search, filters, sorting and pages |
| GET    | `/api/students/stats`       | Count students in total, active, and per course |
| GET    | `/api/students/:id`         | Get one student                                |
| POST   | `/api/students`             | Create a student                               |
| PUT    | `/api/students/:id`         | Replace a student (every field)                |
| PATCH  | `/api/students/:id`         | Change some fields                             |
| DELETE | `/api/students/:id`         | Delete a student                               |

All data will be stored in MongoDB using Mongoose (Session 19)

Search and filters use the query string, like `GET /api/students?search=john&course=Physics&page=2`. In Session 15 you learned that filtering a collection belongs in the query string, not in new URLs like `/students/search`.

This is what a real backend API looks like

---

## Project Structure

Until now, everything was in one server.js. A real project splits the code into folders

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
├── middleware/
│   └── errorHandler.js
│
├── .env
├── .gitignore
├── seed.js
├── server.js
├── test-api.js
└── package.json
```

Why this structure

| Folder         | Holds                         | Answers the question                  |
| -------------- | ----------------------------- | ------------------------------------- |
| `models/`      | Mongoose schemas and models   | What does a student look like?        |
| `controllers/` | The functions that handle requests | What happens for this request?   |
| `routes/`      | URL + method → controller     | Which function runs for this URL?     |
| `middleware/`  | Code that runs between requests and controllers | What runs for many routes? |
| `server.js`    | Connects everything and starts the app | How does the app start?      |

![One big server.js is split into folders, each with one job](images/20-express-mongodb-crud-api/folder-split.gif)

This is called separation of concerns

Each folder has one responsibility. When something breaks, you know where to look: wrong URL → routes, wrong logic → controller, wrong data rule → model.

Session 21 gives this pattern its official name, MVC.

---

## How a Request Travels Through the Folders

![A request goes from server.js to the router, the controller, the model and the database, then back](images/20-express-mongodb-crud-api/request-journey.gif)

| Step | File                              | What happens                                      |
| ---- | --------------------------------- | ------------------------------------------------- |
| 1    | server.js                         | `express.json()` reads the body                   |
| 2    | server.js                         | `app.use("/api/students", studentRoutes)` hands it to the router |
| 3    | routes/studentRoutes.js           | `/:id` + GET → `getStudentById`                   |
| 4    | controllers/studentController.js  | Calls `Student.findById(id)`                      |
| 5    | models/Student.js                 | Mongoose asks MongoDB                             |
| 6    | controller                        | Sends the JSON response                           |
| -    | middleware/errorHandler.js        | Only if something threw an error                  |

Each file `require`s the next one with a relative path (Session 04)

| In this file                         | Require                                   | Why `../`                      |
| ------------------------------------ | ----------------------------------------- | ------------------------------ |
| server.js                            | `require("./routes/studentRoutes")`       | routes is inside the same folder |
| routes/studentRoutes.js              | `require("../controllers/studentController")` | Go up out of routes/ first |
| controllers/studentController.js     | `require("../models/Student")`            | Go up out of controllers/ first |

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

For a local MongoDB, use `MONGODB_URI=mongodb://127.0.0.1:27017` (Session 17)

Create .gitignore

```text
node_modules/
.env
```

Create the folders

```bash
mkdir models controllers routes middleware
```

Add scripts to package.json

```json
{
  "scripts": {
    "start": "node server.js",
    "dev": "node --watch server.js",
    "seed": "node seed.js"
  }
}
```

Now let us create each file

---

## Creating the Student Model

Create models/Student.js

```javascript
const mongoose = require("mongoose");

const studentSchema = new mongoose.Schema(
  {
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
      max: [60, "Age cannot exceed 60"],
      cast: "Age must be a number"
    },
    course: {
      type: String,
      required: [true, "Course name is required"],
      enum: {
        values: ["Computer Science", "Mathematics", "Physics", "Chemistry", "Biology", "Engineering"],
        message: "{VALUE} is not a valid course"
      }
    },
    email: {
      type: String,
      required: [true, "Email is required"],
      unique: true,
      lowercase: true,
      trim: true,
      match: [/^\S+@\S+\.\S+$/, "Please enter a valid email"]
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
  },
  {
    timestamps: true
  }
);

module.exports = mongoose.model("Student", studentSchema);
```

New things compared to Session 19

| Field         | What is new                                                        |
| ------------- | ------------------------------------------------------------------ |
| `address`     | A **nested object**. It is stored inside the student document (Session 17) |
| `phoneNumber` | A String, not a Number. Phone numbers can start with 0, and you never do math with them |
| `isActive`    | A Boolean with a default. Instead of deleting old students, you can mark them inactive |

Why such a simple email pattern? Many tutorials use a long pattern like `/^\w+([\.-]?\w+)*@\w+([\.-]?\w+)*(\.\w{2,3})+$/`. We tested it, and it has two real problems

![A complicated email regex freezes the whole server](images/20-express-mongodb-crud-api/regex-freeze.gif)

| Input                          | Long pattern            | Simple pattern |
| ------------------------------ | ----------------------- | -------------- |
| `john@example.com`             | valid                   | valid          |
| `a@b.info`                     | rejected (4-letter ending) | valid       |
| `sam+news@gmail.com`           | rejected (`+`)          | valid          |
| 31 letters followed by `!`     | **1.9 seconds** to decide | 0 ms         |

The last row is the dangerous one. The long pattern tries millions of combinations before giving up. Node.js has one main thread (Session 03), so for those 1.9 seconds **every** user of your server waits. An attacker could send many such emails and freeze the server. This is called ReDoS (Regular expression Denial of Service). Simple patterns are safer. Real email checking is done by sending a confirmation email.

---

## Creating Controllers

Controllers contain the logic for each request. Each controller is an `async` function with `(req, res)`, exactly like the route handlers you wrote before, just moved into their own file.

Create controllers/studentController.js. We build it in parts. First the helpers and the simple controllers

```javascript
const Student = require("../models/Student");

// Fields a client is allowed to send
const ALLOWED_FIELDS = ["name", "age", "course", "email", "phoneNumber", "address", "isActive"];

const SORT_OPTIONS = ["name", "-name", "age", "-age", "createdAt", "-createdAt"];

// Copy only the allowed fields from the request body
function pickFields(body) {
  const data = {};
  for (const key of ALLOWED_FIELDS) {
    if (body[key] !== undefined) {
      data[key] = body[key];
    }
  }
  return data;
}
```

`pickFields()` is the same idea as the allowed-keys loop from Session 15, written once and used by create, PUT and PATCH. A client cannot set `createdAt`, `_id` or `__v` (Session 19).

![pickFields lets only the allowed fields through](images/20-express-mongodb-crud-api/pick-fields.gif)

```javascript
// GET /api/students/:id
const getStudentById = async (req, res) => {
  const student = await Student.findById(req.params.id);

  if (!student) {
    return res.status(404).json({ success: false, message: "Student not found" });
  }

  res.status(200).json({ success: true, data: student });
};

// POST /api/students
const createStudent = async (req, res) => {
  const student = await Student.create(pickFields(req.body || {}));

  res.status(201).json({ success: true, data: student });
};

// PUT /api/students/:id - replace every field
const replaceStudent = async (req, res) => {
  const student = await Student.findById(req.params.id);

  if (!student) {
    return res.status(404).json({ success: false, message: "Student not found" });
  }

  student.overwrite(pickFields(req.body || {}));
  await student.save();

  res.status(200).json({ success: true, data: student });
};

// PATCH /api/students/:id - change some fields
const updateStudent = async (req, res) => {
  const student = await Student.findByIdAndUpdate(req.params.id, pickFields(req.body || {}), {
    returnDocument: "after",
    runValidators: true
  });

  if (!student) {
    return res.status(404).json({ success: false, message: "Student not found" });
  }

  res.status(200).json({ success: true, data: student });
};

// DELETE /api/students/:id
const deleteStudent = async (req, res) => {
  const student = await Student.findByIdAndDelete(req.params.id);

  if (!student) {
    return res.status(404).json({ success: false, message: "Student not found" });
  }

  res.status(200).json({ success: true, message: "Student deleted successfully" });
};
```

Things to notice

| Code                          | Why                                                          |
| ----------------------------- | ------------------------------------------------------------ |
| No try / catch                | Express 5 sends errors from async functions to the error handler (Session 15). Validation, bad ids and duplicates are all handled there |
| `student.overwrite(...)`      | Replaces all fields with the new ones. We tested it: it keeps `_id` and `createdAt`, puts back defaults like `isActive: true`, and removes fields that were not sent (like `address`) |
| `await student.save()`        | Runs every schema rule (Session 19)                          |
| `runValidators: true`         | PATCH must follow the schema too                             |
| `returnDocument: "after"`     | Send the student after the change (not `{ new: true }`, which is deprecated in Mongoose 9) |

![PATCH with a nested object replaces the whole address](images/20-express-mongodb-crud-api/nested-patch.gif)

Watch out with nested objects in PATCH. Sending `{ "address": { "city": "LA" } }` replaces the **whole** address: we tested it, and `street` and `zipCode` were gone afterwards. The client must send the complete address. (MongoDB can change one nested field with dot notation, `{ "address.city": "LA" }`, but our `pickFields` only allows `address`, which keeps things simple.)

---

## Searching, Filtering and Sorting

The list controller builds one filter object from the query string, like Session 18, but with more options

```javascript
// Put a \ before characters that have a special meaning in a regex
function escapeRegex(text) {
  return text.replace(/[.*+?^${}()|[\]\\]/g, "\\$&");
}
```

```javascript
// GET /api/students?search=&course=&city=&isActive=&sort=&page=&limit=
const getAllStudents = async (req, res) => {
  const filter = {};

  if (req.query.search) {
    filter.name = { $regex: escapeRegex(req.query.search), $options: "i" };
  }
  if (req.query.course) {
    filter.course = req.query.course;
  }
  if (req.query.city) {
    filter["address.city"] = req.query.city;
  }
  if (req.query.isActive) {
    filter.isActive = req.query.isActive === "true";
  }

  const sort = req.query.sort || "-createdAt";
  if (!SORT_OPTIONS.includes(sort)) {
    return res.status(400).json({ success: false, message: `sort must be one of: ${SORT_OPTIONS.join(", ")}` });
  }

  // ... pagination, see the next section
};
```

![Each query parameter adds one condition to the filter](images/20-express-mongodb-crud-api/query-builder.gif)

| URL                                  | filter                                        |
| ------------------------------------ | --------------------------------------------- |
| `?search=john`                       | `{ name: { $regex: "john", $options: "i" } }` |
| `?course=Physics`                    | `{ course: "Physics" }`                       |
| `?city=Boston`                       | `{ "address.city": "Boston" }`                |
| `?isActive=false`                    | `{ isActive: false }`                         |
| `?search=jo&city=New York`           | both conditions (AND)                         |

New ideas

* **Search with `$regex`** (Session 18). `$options: "i"` ignores capital letters, so `john` finds "John Doe", "Johnny Lee" and "Mike Johnson".
* **Escaping the search text.** Characters like `(`, `*` and `.` have special meanings in a regex. Without `escapeRegex()`, searching for `(` crashes with a 500 ("Regular expression is invalid"), and a clever search text could make the database work very hard. `escapeRegex()` puts a `\` before each special character, so `(` is searched as a normal character. You do not need to understand the pattern inside it; copy it as it is.
* **Searching inside a nested object** uses dot notation in quotes: `"address.city"`. Bracket notation `filter["address.city"]` is needed because of the dot.
* **Booleans from the query string.** `req.query.isActive` is the string `"false"`, which is truthy (Session 16). `=== "true"` turns it into a real boolean.
* **Sort allow list.** Like Session 19, only known values are accepted.

---

## Pagination

With 10,000 students, sending all of them in one response is slow for the server, the network and the phone that shows them. **Pagination** sends one page at a time. You built it with an array and `slice()` in Session 13; here MongoDB does the work with `skip()` and `limit()` (Session 18).

The rest of getAllStudents

```javascript
  let page = parseInt(req.query.page) || 1;
  let limit = parseInt(req.query.limit) || 10;
  if (page < 1) page = 1;
  if (limit < 1) limit = 10;
  if (limit > 100) limit = 100;

  const students = await Student.find(filter)
    .sort(`${sort} _id`) // _id breaks ties, so pages never mix up
    .skip((page - 1) * limit)
    .limit(limit);

  const total = await Student.countDocuments(filter);

  res.status(200).json({
    success: true,
    count: students.length,
    total,
    page,
    totalPages: Math.ceil(total / limit),
    data: students
  });
};
```

![skip jumps over earlier pages, limit takes one page](images/20-express-mongodb-crud-api/pagination.gif)

| page | limit | skip `(page - 1) * limit` | Students returned |
| ---- | ----- | ------------------------- | ----------------- |
| 1    | 3     | 0                         | 1st to 3rd        |
| 2    | 3     | 3                         | 4th to 6th        |
| 3    | 3     | 6                         | 7th to 9th        |

| Code                          | Why                                                      |
| ----------------------------- | -------------------------------------------------------- |
| `parseInt(req.query.page)`    | Turns `"2"` into `2`. `parseInt("abc")` is `NaN`, so `\|\| 1` gives page 1. `parseInt("2.7")` is `2`, a whole number |
| `if (limit > 100) limit = 100` | A client cannot ask for a million students at once      |
| `countDocuments(filter)`      | The total for the **same filter**, so totalPages is correct |
| `Math.ceil(total / limit)`    | 8 students with limit 3 is 3 pages (Session 13)          |
| `` `${sort} _id` ``           | Sort by `_id` too when the main field is equal           |

Why `_id` as a tie-breaker? `insertMany()` gives many students the same `createdAt`. When two students have the same sort value, MongoDB may return them in a different order on each request, so a student could appear on page 1 and again on page 2, while another is never shown. Adding `_id` (always unique) makes the order fixed. `"-createdAt _id"` means "newest first, then by `_id`".

Example response for `GET /api/students?sort=name&page=2&limit=3` with 8 students

```json
{
  "success": true,
  "count": 3,
  "total": 8,
  "page": 2,
  "totalPages": 3,
  "data": [ "... John Doe, Johnny Lee, Mike Johnson ..." ]
}
```

Statistics: one more controller

```javascript
// GET /api/students/stats
const getStats = async (req, res) => {
  const total = await Student.countDocuments();
  const active = await Student.countDocuments({ isActive: true });

  const byCourse = {};
  const courses = await Student.distinct("course");
  for (const course of courses) {
    byCourse[course] = await Student.countDocuments({ course });
  }

  res.status(200).json({ success: true, data: { total, active, inactive: total - active, byCourse } });
};
```

`distinct("course")` returns each different course name once, like `["Biology", "Computer Science", ...]`. `{ course }` is short for `{ course: course }`.

At the end of the controller file, export everything

```javascript
module.exports = {
  getAllStudents,
  getStats,
  getStudentById,
  createStudent,
  replaceStudent,
  updateStudent,
  deleteStudent
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
  getStats,
  getStudentById,
  createStudent,
  replaceStudent,
  updateStudent,
  deleteStudent
} = require("../controllers/studentController");

// Fixed paths first, before /:id
router.get("/stats", getStats);

router.route("/")
  .get(getAllStudents)
  .post(createStudent);

router.route("/:id")
  .get(getStudentById)
  .put(replaceStudent)
  .patch(updateStudent)
  .delete(deleteStudent);

module.exports = router;
```

`router.route()` (Session 13) groups all methods for the same path. The routes file is now a short table of contents for the API: you can see every endpoint without reading any logic.

Notice the order of routes

![/stats must come before /:id, or "stats" is treated as an id](images/20-express-mongodb-crud-api/route-order.gif)

`/stats` comes before `/:id`. If `/:id` came first, `GET /api/students/stats` would match `/:id` with `id = "stats"`, and `findById("stats")` would give a CastError: "Invalid _id: stats".

---

## The Error Handler Middleware

The error handler from Session 19 moves into its own file

Create middleware/errorHandler.js

```javascript
// 404 for routes that do not exist
function notFound(req, res) {
  res.status(404).json({ success: false, message: `Route ${req.method} ${req.originalUrl} not found` });
}

// One place that turns every error into a clean JSON answer
function errorHandler(err, req, res, next) {
  // The body was not valid JSON
  if (err.type === "entity.parse.failed") {
    return res.status(400).json({ success: false, message: "Request body is not valid JSON" });
  }

  // Schema rules failed
  if (err.name === "ValidationError") {
    const errors = Object.values(err.errors).map((e) => e.message);
    return res.status(400).json({ success: false, message: "Validation failed", errors });
  }

  // Wrong type, for example a bad id
  if (err.name === "CastError") {
    return res.status(400).json({ success: false, message: `Invalid ${err.path}: ${err.value}` });
  }

  // Unique index: the email already exists
  if (err.code === 11000) {
    return res.status(409).json({ success: false, message: "Email already exists" });
  }

  console.error(err);
  res.status(500).json({ success: false, message: "Something went wrong on the server" });
}

module.exports = { notFound, errorHandler };
```

`errorHandler` must keep all four parameters `(err, req, res, next)`, even though `next` is not used. With three parameters, Express does not treat it as an error handler (Session 14).

`req.originalUrl` is the full URL, like `/api/students/abc/xyz`. Inside a router, `req.url` only has the part after the mount path.

---

## Putting It All Together

Create server.js

```javascript
require("dotenv").config({ quiet: true });
const express = require("express");
const mongoose = require("mongoose");

const studentRoutes = require("./routes/studentRoutes");
const { notFound, errorHandler } = require("./middleware/errorHandler");

const app = express();
const PORT = process.env.PORT || 5000;

// Middleware
app.use(express.json());

// Home route: a short guide to the API
app.get("/", (req, res) => {
  res.json({
    message: "Welcome to Student Management API",
    endpoints: {
      listStudents: "GET /api/students?search=&course=&city=&isActive=&sort=&page=&limit=",
      stats: "GET /api/students/stats",
      getStudent: "GET /api/students/:id",
      createStudent: "POST /api/students",
      replaceStudent: "PUT /api/students/:id",
      updateStudent: "PATCH /api/students/:id",
      deleteStudent: "DELETE /api/students/:id"
    }
  });
});

// Routes
app.use("/api/students", studentRoutes);

// 404 and errors (always last)
app.use(notFound);
app.use(errorHandler);

// Connect to MongoDB, then start the server
async function startServer() {
  try {
    await mongoose.connect(process.env.MONGODB_URI, { dbName: process.env.DB_NAME });
    console.log("Connected to MongoDB");
  } catch (error) {
    console.error("MongoDB connection failed:", error.message);
    process.exit(1);
  }

  app.listen(PORT, (err) => {
    if (err) {
      console.log("Could not start server:", err.message);
      return;
    }

    console.log(`Server running on port ${PORT}`);
    console.log(`http://localhost:${PORT}`);
  });
}

startServer();
```

server.js is now short. It only sets things up: middleware, routes, error handling, database, listen.

| Old code you may see in tutorials            | Problem                                                   | This lesson                        |
| -------------------------------------------- | --------------------------------------------------------- | ---------------------------------- |
| `app.use("*", ...)`                          | Crashes at startup in Express 5 (Session 12)              | `app.use(notFound)`                |
| `` `${MONGODB_URI}/${DB_NAME}` ``            | Silently uses the wrong database (Session 19)             | `{ dbName: process.env.DB_NAME }`  |
| `{ new: true }`                              | Deprecated in Mongoose 9, prints a warning                | `returnDocument: "after"`          |
| try / catch with `error: error.message` in every controller | Repeats code and shows internal errors to users | One error handler                |
| `Student.create(req.body)`                   | Client can set any schema field                           | `pickFields(req.body)`             |

---

## Adding Sample Data

seed.js

```javascript
require("dotenv").config({ quiet: true });
const mongoose = require("mongoose");
const Student = require("./models/Student");

const students = [
  { name: "John Doe", age: 20, course: "Computer Science", email: "john@example.com", phoneNumber: "1234567890", address: { street: "123 Main St", city: "New York", zipCode: "10001" } },
  { name: "Jane Smith", age: 22, course: "Mathematics", email: "jane@example.com", phoneNumber: "2345678901", address: { city: "Boston" } },
  { name: "Mike Johnson", age: 21, course: "Physics", email: "mike@example.com", phoneNumber: "3456789012", address: { city: "New York" } },
  { name: "Sara Khan", age: 24, course: "Computer Science", email: "sara@example.com", phoneNumber: "4567890123", address: { city: "Chicago" } },
  { name: "Tom Brown", age: 19, course: "Mathematics", email: "tom@example.com", phoneNumber: "5678901234", address: { city: "Boston" }, isActive: false },
  { name: "Emma Wilson", age: 23, course: "Biology", email: "emma@example.com", phoneNumber: "6789012345", address: { city: "Chicago" } },
  { name: "Johnny Lee", age: 26, course: "Engineering", email: "johnny@example.com", phoneNumber: "7890123456", address: { city: "New York" } }
];

async function seed() {
  try {
    await mongoose.connect(process.env.MONGODB_URI, { dbName: process.env.DB_NAME });

    await Student.deleteMany({});
    const created = await Student.insertMany(students);
    console.log("Added students:", created.length);
  } catch (error) {
    console.error("Seeding failed:", error.message);
  } finally {
    await mongoose.disconnect();
  }
}

seed();
```

```bash
npm run seed
```

Output

```text
Added students: 7
```

---

## Testing the API

Start the server

```bash
npm run dev
```

You will see

```text
Connected to MongoDB
Server running on port 5000
http://localhost:5000
```

GET requests work in the browser

```text
http://localhost:5000/
http://localhost:5000/api/students
http://localhost:5000/api/students?search=john
http://localhost:5000/api/students?city=New%20York&sort=-age
http://localhost:5000/api/students?sort=name&page=2&limit=3
http://localhost:5000/api/students/stats
```

`%20` is a space in a URL. Browsers add it for you when you type a space.

For everything else, this test script checks every route and prints one line per request

test-api.js

```javascript
const BASE = "http://localhost:5000/api/students";

async function send(method, path, body) {
  const res = await fetch(BASE + path, {
    method,
    headers: { "Content-Type": "application/json" },
    body: body ? JSON.stringify(body) : undefined
  });
  const data = await res.json();
  return { status: res.status, data };
}

// Show only the useful part of each answer
function show(label, { status, data }) {
  let info = data.message || "";
  if (data.errors) info += " " + JSON.stringify(data.errors);
  if (Array.isArray(data.data)) info += ` ${data.count} of ${data.total}: ` + data.data.map((s) => s.name).join(", ");
  else if (data.data && data.data.name) info += ` ${data.data.name}, age ${data.data.age}, ${data.data.course}`;
  else if (data.data) info += " " + JSON.stringify(data.data);
  console.log(`${label.padEnd(34)} ${status} ${info.trim()}`);
}

async function test() {
  const created = await send("POST", "", {
    name: "Ali Raza", age: 21, course: "Physics", email: "ali@example.com",
    phoneNumber: "9876543210", address: { city: "Lahore" }
  });
  show("POST new student", created);
  const id = created.data.data._id;

  show("POST invalid", await send("POST", "", { name: "A", age: 15, course: "Art", email: "bad", phoneNumber: "123" }));
  show("POST same email", await send("POST", "", { name: "Copy", age: 30, course: "Physics", email: "ali@example.com", phoneNumber: "1112223334" }));
  show("GET ?search=john", await send("GET", "?search=john"));
  show("GET ?course=Mathematics", await send("GET", "?course=Mathematics"));
  show("GET ?city=New York&sort=-age", await send("GET", "?city=New%20York&sort=-age"));
  show("GET ?isActive=false", await send("GET", "?isActive=false"));
  show("GET ?sort=name&page=2&limit=3", await send("GET", "?sort=name&page=2&limit=3"));
  show("GET ?sort=password", await send("GET", "?sort=password"));
  show("GET ?search=(", await send("GET", "?search=("));
  show("GET /stats", await send("GET", "/stats"));
  show("PATCH age", await send("PATCH", "/" + id, { age: 22 }));
  show("PATCH bad course", await send("PATCH", "/" + id, { course: "Art" }));
  show("PUT missing fields", await send("PUT", "/" + id, { name: "Ali R" }));
  show("GET /123", await send("GET", "/123"));
  show("DELETE", await send("DELETE", "/" + id));
  show("GET deleted", await send("GET", "/" + id));
}

test();
```

`padEnd(34)` adds spaces to the end of the label until it is 34 characters long, so the status codes line up in a column.

Run it in a second terminal, after `npm run seed`

```bash
node test-api.js
```

Output

```text
POST new student                   201 Ali Raza, age 21, Physics
POST invalid                       400 Validation failed ["Name must be at least 2 characters","Age must be at least 18","Art is not a valid course","Please enter a valid email","Phone number must be 10 digits"]
POST same email                    409 Email already exists
GET ?search=john                   200 3 of 3: Mike Johnson, Johnny Lee, John Doe
GET ?course=Mathematics            200 2 of 2: Jane Smith, Tom Brown
GET ?city=New York&sort=-age       200 3 of 3: Johnny Lee, Mike Johnson, John Doe
GET ?isActive=false                200 1 of 1: Tom Brown
GET ?sort=name&page=2&limit=3      200 3 of 8: John Doe, Johnny Lee, Mike Johnson
GET ?sort=password                 400 sort must be one of: name, -name, age, -age, createdAt, -createdAt
GET ?search=(                      200 0 of 0:
GET /stats                         200 {"total":8,"active":7,"inactive":1,"byCourse":{"Biology":1,"Computer Science":2,"Engineering":1,"Mathematics":2,"Physics":2}}
PATCH age                          200 Ali Raza, age 22, Physics
PATCH bad course                   400 Validation failed ["Art is not a valid course"]
PUT missing fields                 400 Validation failed ["Student age is required","Course name is required","Email is required","Phone number is required"]
GET /123                           400 Invalid _id: 123
DELETE                             200 Student deleted successfully
GET deleted                        404 Student not found
```

![The test script checks every route and status code](images/20-express-mongodb-crud-api/test-run.gif)

Things to check in the output

* `search=john` finds 3 students, including "Mike **John**son", because the search looks anywhere in the name.
* Page 2 of the name-sorted list is students 4 to 6 out of 8 (7 from the seed + Ali).
* `search=(` gives 0 results instead of a 500, thanks to `escapeRegex()`.
* `/stats` is not treated as an id, because its route comes first.

curl on macOS, Linux or Git Bash

```bash
curl -X POST http://localhost:5000/api/students -H "Content-Type: application/json" -d '{"name":"John Doe","age":20,"course":"Computer Science","email":"john2@example.com","phoneNumber":"1234567890","address":{"city":"New York"}}'

curl "http://localhost:5000/api/students?search=john"

curl -X PATCH http://localhost:5000/api/students/STUDENT_ID_HERE -H "Content-Type: application/json" -d '{"age":21}'

curl -X DELETE http://localhost:5000/api/students/STUDENT_ID_HERE
```

On Windows, use the Command Prompt or PowerShell commands from [Session 11](11-crud-operations-dummy-data.md#testing-your-api), or run test-api.js.

---

## Beginner Mistakes

### Mistake 1

Wrong relative path in require.

`require("./models/Student")` inside controllers/ looks for controllers/models/Student.js and fails with "Cannot find module". From a subfolder, go up first: `require("../models/Student")`.

---

### Mistake 2

Forgetting to export a controller, or a typo in its name.

If `getStats` is not in `module.exports`, the destructured `getStats` is `undefined`, and Express crashes at startup with "argument handler must be a function" (Session 13).

---

### Mistake 3

Putting `/:id` before fixed paths like `/stats`.

"stats" becomes the id and you get a CastError.

---

### Mistake 4

Pagination without a maximum limit.

`?limit=1000000` would load every student into memory. Always cap it.

---

### Mistake 5

Counting with a different filter than the list.

`countDocuments()` without the filter gives the total of all students, and `totalPages` is wrong when a search is active.

---

### Mistake 6

Putting user text directly into `$regex`.

`?search=(` crashes with a 500, and complex patterns can slow the database. Escape the text first.

---

### Mistake 7

Using complicated regex patterns for validation.

A pattern with nested repeats like `(\w+)*` can take seconds on some inputs and freeze the whole server (ReDoS).

---

### Mistake 8

Sending part of a nested object in PATCH.

`{ "address": { "city": "LA" } }` replaces the whole address. Send the full object.

---

### Mistake 9

Comparing a query string to a boolean.

`filter.isActive = req.query.isActive` stores the string `"false"`. Use `req.query.isActive === "true"`.

---

## Practice Exercises

### Exercise 1

Add a new field to the Student model

```text
grade: A, B, C, D, F
```

Use `enum` for validation. Add `grade` to `ALLOWED_FIELDS`

### Exercise 2

Add a `?grade=A` filter to GET /api/students

### Exercise 3

Add `?minAge=` and `?maxAge=` filters with `$gte` and `$lte` (Session 18). Return 400 if they are not numbers

### Exercise 4

Add `PATCH /api/students/:id/deactivate` that sets `isActive` to false. Where must this route go in the routes file, and why does it not clash with `/:id`?

### Exercise 5

Create a Course model with

* name
* description
* duration (weeks)
* price

Create courseController.js and courseRoutes.js with full CRUD, and mount them at `/api/courses`

### Exercise 6

Add a relationship between Student and Course

Each student can enroll in multiple courses: `courses: [{ type: mongoose.Schema.Types.ObjectId, ref: "Course" }]`. Use `populate("courses", "name")` in getStudentById (Session 19)

### Exercise 7

Add `hasNextPage` and `hasPrevPage` (true or false) to the list response

### Exercise 8

Change the home route so it also shows how many students are in the database

---

## Interview Questions

### Why do we separate routes, controllers, and models

To keep code organized and maintainable. Each folder has one responsibility (separation of concerns), so code is easier to find, test and change

### What is the purpose of controllers

Controllers contain the logic for handling requests: read input, call the model, send the response

### What is the purpose of routes

Routes define the API endpoints and connect each URL and method to a controller

### Why is route order important

Express matches routes in order, so fixed paths like /stats must come before parameter routes like /:id

### What is pagination and why do we use it

Pagination splits large result sets into pages with skip and limit. It keeps responses small and fast

### How do you calculate skip for pagination

skip = (page - 1) * limit

### Why should you cap the limit

So a client cannot request millions of documents in one response

### Why add _id to the sort

Documents with equal sort values may come back in a different order on each request. _id is unique, so the order and the pages stay stable

### What does $regex do in MongoDB

$regex allows pattern matching for string searches

### What does the option $options: "i" do

It makes the search case insensitive

### Why escape user input before using it in $regex

Special characters like ( or * would change the pattern. An invalid pattern causes an error, and a complex one can slow down the database

### What is ReDoS

Regular expression Denial of Service. A badly written regex takes a very long time on certain inputs, which blocks Node.js's single thread and freezes the server

### How do you query a field inside a nested object

With dot notation in quotes, for example { "address.city": "Boston" }

### What is the difference between PUT and PATCH in this API

PUT replaces all fields (overwrite + save). PATCH changes only the fields sent (findByIdAndUpdate with runValidators)

---
