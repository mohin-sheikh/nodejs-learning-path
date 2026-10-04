## Table of Contents

* [What is MVC](#what-is-mvc)
* [Why Use MVC](#why-use-mvc)
* [MVC Components Explained](#mvc-components-explained)
* [You Already Built MVC](#you-already-built-mvc)
* [Fat Model, Thin Controller](#fat-model-thin-controller)
* [Setting Up MVC Project](#setting-up-mvc-project)
* [Model Layer](#model-layer)
* [View Layer](#view-layer)
* [EJS Tags](#ejs-tags)
* [Partials: Reusing Header and Footer](#partials-reusing-header-and-footer)
* [Escaping and XSS](#escaping-and-xss)
* [Controller Layer](#controller-layer)
* [Forms and Post/Redirect/Get](#forms-and-postredirectget)
* [Routes for Pages and API](#routes-for-pages-and-api)
* [Error Pages for Browsers](#error-pages-for-browsers)
* [Complete MVC Example](#complete-mvc-example)
* [How MVC Works Together](#how-mvc-works-together)
* [Beginner Mistakes](#beginner-mistakes)
* [Practice Exercises](#practice-exercises)
* [Interview Questions](#interview-questions)

---

## What is MVC

MVC is a design pattern

It stands for Model View Controller

It helps organize your code into three separate parts

Think of it like a restaurant

| Restaurant      | MVC         | Job                                      |
| --------------- | ----------- | ---------------------------------------- |
| Kitchen         | Model       | Prepares the food (the data)             |
| Plate on the table | View     | Presents the food to the customer        |
| Waiter          | Controller  | Takes the order, talks to the kitchen, brings the plate |

![The waiter (controller) takes the order to the kitchen (model) and brings back the plate (view)](images/21-mvc-architecture/restaurant.gif)

When a customer comes to a restaurant

1. The customer tells the order to the waiter (Controller)
2. The waiter takes the order to the kitchen (Model)
3. The kitchen prepares the food (Model)
4. The waiter puts the food on a plate (View)
5. The customer sees the food (View)

The customer never walks into the kitchen, and the cook never talks to the customer.

Same in web applications

1. The browser sends a request to the Controller
2. The Controller asks the Model for data
3. The Model gets data from the database
4. The Controller passes the data to the View
5. The View turns it into HTML (or JSON) for the user

---

## Why Use MVC

Problems without MVC

* All code in one file
* Hard to find where things are
* A change in one place breaks another
* Cannot reuse code
* Hard to work in teams: everyone edits the same file

Example of bad code without MVC

```javascript
// Everything mixed together
app.get("/students", async (req, res) => {
  // Database logic here
  const students = await Student.find();

  // Business logic here
  const activeStudents = students.filter((s) => s.isActive);

  // HTML generation here
  let html = "<html><body>";
  activeStudents.forEach((s) => {
    html += `<p>${s.name}</p>`;
  });
  html += "</body></html>";

  res.send(html);
});
```

![The mixed route is split into model, controller and view](images/21-mvc-architecture/split-mixed-code.gif)

| Part of the code                    | Where it belongs in MVC                         |
| ----------------------------------- | ----------------------------------------------- |
| `Student.find()`, "who is active"   | Model                                           |
| Receive the request, choose what to do | Controller                                   |
| Building HTML with `html +=`        | View (a template file)                          |

Building HTML with `+=` also has a hidden security bug: a student named `<script>...</script>` would run JavaScript in every visitor's browser. Templates fix this, see [Escaping and XSS](#escaping-and-xss).

Benefits of MVC

* Separation of concerns: each file has one job
* Easy to find and fix bugs
* The same model can be used by an HTML page and a JSON API
* A designer can change views without touching database code
* Several developers can work at the same time without conflicts

---

## MVC Components Explained

### Model

The Model handles data and database operations

| Question               | Answer                                         |
| ---------------------- | ---------------------------------------------- |
| What it does           | Talks to the database, enforces data rules     |
| What it contains       | Schema, validation, methods, statics           |
| What it does NOT do    | Read `req`, send responses, build HTML         |

Example of Model work: find all students, save a new student, check that an email is valid, build a student's summary text.

### View

The View handles what the user sees

| Question               | Answer                                         |
| ---------------------- | ---------------------------------------------- |
| What it does           | Shows data to the user                         |
| What it contains       | HTML templates (EJS), or the JSON shape of an API |
| What it does NOT do    | Talk to the database, decide what happens      |

Example of View work: show the student list in HTML, show a form, show error messages.

### Controller

The Controller handles the request

| Question               | Answer                                         |
| ---------------------- | ---------------------------------------------- |
| What it does           | Reads the request, calls the model, chooses the view |
| What it contains       | `async (req, res) => { ... }` functions        |
| What it does NOT do    | Contain schema rules, build HTML by hand       |

Example of Controller work: receive `GET /students`, ask the model for students, pass them to the list view.

---

## You Already Built MVC

Session 20 already used this pattern, without the name

| Session 20 folder        | MVC part                                   |
| ------------------------ | ------------------------------------------ |
| `models/Student.js`      | Model                                      |
| `controllers/studentController.js` | Controller                       |
| `res.json({ success, data })` | View: for an API, the view is the JSON |
| `routes/studentRoutes.js` | Not a letter in MVC. Routes send each URL to the right controller |
| `middleware/errorHandler.js` | Shared helper used by all controllers  |

In an API, the "view" is simply the JSON you send. In a website, the view is an HTML page. This session adds real HTML views with EJS, next to the JSON API, so you can see that **the same model serves both**.

---

## Fat Model, Thin Controller

A common rule: put logic about the **data** in the model, and keep controllers short.

![Data logic moves into the model, the controller stays short](images/21-mvc-architecture/fat-model.gif)

Thin controller (good)

```javascript
const students = await Student.findByCourse(req.query.course);
```

Fat controller (avoid)

```javascript
const students = await Student.find({ course: req.query.course }).sort("name");
```

The first version hides the details in the model. If the rule changes ("only active students", "also sort by grade"), you change it in one place, and every controller that uses `findByCourse` gets the change.

| Put it in the model                         | Put it in the controller                       |
| ------------------------------------------- | ---------------------------------------------- |
| Validation rules                            | Reading `req.params`, `req.query`, `req.body`  |
| Reusable queries (`findByCourse`)           | Choosing the status code                       |
| Text built from a document (`getSummary`)   | Choosing JSON or which view to render          |
| Hooks like hashing a password (Session 23)  | Redirecting after a form                       |

---

## Setting Up MVC Project

Create a new project

```bash
mkdir mvc-app
cd mvc-app
npm init -y
```

Install packages

```bash
npm install express mongoose dotenv ejs
```

Create folder structure

```bash
mkdir models controllers routes middleware views views/partials views/students
```

Create .env file

```text
PORT=5000
MONGODB_URI=mongodb+srv://yourusername:yourpassword@cluster0.abc123.mongodb.net/
DB_NAME=mvc_school
```

For a local MongoDB, use `MONGODB_URI=mongodb://127.0.0.1:27017` (Session 17)

Final structure

```text
mvc-app/
├── models/
│   └── Student.js              Model
├── views/                      Views
│   ├── partials/
│   │   ├── header.ejs
│   │   └── footer.ejs
│   ├── students/
│   │   ├── list.ejs
│   │   ├── detail.ejs
│   │   └── new.ejs
│   └── error.ejs
├── controllers/                Controllers
│   ├── studentController.js    JSON API
│   └── pageController.js       HTML pages
├── routes/
│   ├── studentRoutes.js        /api/students
│   └── pageRoutes.js           /students
├── middleware/
│   └── errorHandler.js
├── .env
├── seed.js
└── server.js
```

Now let us build each layer

---

## Model Layer

The Model layer is responsible for data

Create models/Student.js

```javascript
const mongoose = require("mongoose");

const studentSchema = new mongoose.Schema(
  {
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
      max: [60, "Age cannot exceed 60"],
      cast: "Age must be a number"
    },
    course: {
      type: String,
      required: [true, "Course is required"],
      trim: true
    },
    email: {
      type: String,
      required: [true, "Email is required"],
      unique: true,
      lowercase: true,
      trim: true,
      match: [/^\S+@\S+\.\S+$/, "Please enter a valid email"]
    },
    grade: {
      type: String,
      enum: { values: ["A", "B", "C", "D", "F"], message: "{VALUE} is not a valid grade" },
      default: "B"
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

// Instance method: works on ONE student (this = the student)
studentSchema.methods.getSummary = function () {
  return `${this.name} is studying ${this.course} and got grade ${this.grade}`;
};

// Static method: works on the WHOLE collection (this = the Student model)
studentSchema.statics.findByCourse = function (courseName) {
  return this.find({ course: courseName }).sort("name");
};

module.exports = mongoose.model("Student", studentSchema);
```

Instance methods vs static methods

![A method is called on one student, a static is called on the Student model](images/21-mvc-architecture/methods-vs-statics.gif)

| Kind              | Defined with          | Called on             | `this` is       | Example                         |
| ----------------- | --------------------- | --------------------- | --------------- | ------------------------------- |
| Instance method   | `schema.methods.x`    | One document          | That document   | `john.getSummary()`             |
| Static method     | `schema.statics.x`    | The model             | The model       | `Student.findByCourse("Physics")` |

```javascript
const john = await Student.findOne({ email: "john@example.com" });
console.log(john.getSummary());

const csStudents = await Student.findByCourse("Computer Science");
console.log(csStudents.map((s) => s.name));
```

Output

```text
John Doe is studying Computer Science and got grade A
[ 'John Doe', 'Sara Khan' ]
```

Ask yourself: "Do I need one student first?" Yes → instance method. No, I am searching or counting → static.

Both use `function () { }`, not an arrow function, so `this` works (Session 19).

---

## View Layer

The View layer is what the user sees

For APIs, the View is usually JSON. For a website, the view is an HTML page. We build HTML with a **template engine**.

A template is an HTML file with holes in it. The template engine fills the holes with data and produces normal HTML.

![The template and the data go into EJS, and an HTML page comes out](images/21-mvc-architecture/template-render.gif)

We use EJS (Embedded JavaScript). It is plain HTML plus a few tags.

Tell Express to use EJS in server.js

```javascript
const path = require("path");

app.set("view engine", "ejs");
app.set("views", path.join(__dirname, "views"));
```

| Line                                     | Meaning                                          |
| ---------------------------------------- | ------------------------------------------------ |
| `app.set("view engine", "ejs")`          | Files in views/ end with `.ejs`, you can leave out the ending |
| `app.set("views", path.join(__dirname, "views"))` | Where the view files are (Session 06)   |

Then a controller renders a view with `res.render()`

```javascript
res.render("students/list", { title: "All Students", students });
```

| Part                                     | Meaning                                          |
| ---------------------------------------- | ------------------------------------------------ |
| `"students/list"`                        | The file views/students/list.ejs                 |
| `{ title, students }`                    | Data the template can use as variables           |

`res.render()` fills the template, sets `Content-Type: text/html`, and sends the page.

views/students/list.ejs

```html
<%- include("../partials/header", { title }) %>

<p>Total students: <%= students.length %></p>

<% if (students.length === 0) { %>
  <p>No students yet. <a href="/students/new">Add the first one</a></p>
<% } %>

<% students.forEach((student) => { %>
  <div class="card">
    <a href="/students/<%= student._id %>"><strong><%= student.name %></strong></a>
    <div class="muted">
      Age <%= student.age %> | <%= student.course %> | Grade <%= student.grade %>
    </div>
  </div>
<% }); %>

<%- include("../partials/footer") %>
```

Part of the HTML it produces (tested with the seed data)

```html
<p>Total students: 4</p>

  <div class="card">
    <a href="/students/6ac20fa5f80536d18f084898"><strong>Jane Smith</strong></a>
    <div class="muted">
      Age 22 | Mathematics | Grade B
    </div>
  </div>
```

views/students/detail.ejs

```html
<%- include("../partials/header", { title }) %>

<div class="card">
  <p><%= student.getSummary() %></p>
  <p>Email: <%= student.email %></p>
  <p>Status: <%= student.isActive ? "Active" : "Inactive" %></p>
  <p class="muted">Added on <%= student.createdAt.toDateString() %></p>
</div>

<a href="/students">Back to all students</a>

<%- include("../partials/footer") %>
```

Output

```html
<div class="card">
  <p>Ali Raza is studying Physics and got grade A</p>
  <p>Email: ali@example.com</p>
  <p>Status: Active</p>
  <p class="muted">Added on Sun Oct 04 2026</p>
</div>
```

The view calls the model's `getSummary()` method. A view may **read** data and call simple methods, but it never saves or searches the database.

`toDateString()` is a built-in Date method that gives a short readable date.

---

## EJS Tags

| Tag            | What it does                                   | Example                                  |
| -------------- | ---------------------------------------------- | ---------------------------------------- |
| `<%= value %>` | Print a value, **escaped** (safe)              | `<%= student.name %>`                    |
| `<%- value %>` | Print a value **without** escaping             | `<%- include("../partials/header") %>`   |
| `<% code %>`   | Run JavaScript, print nothing                  | `<% students.forEach((s) => { %>`        |

`<% %>` is used for `if` and loops. Notice that a loop opens in one tag and closes in another:

```html
<% students.forEach((student) => { %>
  ... HTML repeated for each student ...
<% }); %>
```

Everything between the two tags is repeated for each student.

A typo in a variable name stops the page with an error, just like in normal JavaScript

```text
ReferenceError: nmae is not defined
```

---

## Partials: Reusing Header and Footer

Every page needs the same `<head>`, styles and menu. Instead of copying them into each view, put them in **partials** and include them.

views/partials/header.ejs

```html
<!DOCTYPE html>
<html>
<head>
  <title><%= title %> | Student Manager</title>
  <style>
    body { font-family: Arial, sans-serif; margin: 40px; max-width: 700px; }
    nav a { margin-right: 15px; }
    .card { border: 1px solid #ddd; padding: 15px; margin: 10px 0; border-radius: 5px; }
    .muted { color: #666; }
    .errors { background: #fee2e2; color: #991b1b; padding: 10px 25px; border-radius: 5px; }
    label { display: block; margin-top: 10px; }
  </style>
</head>
<body>
  <nav>
    <a href="/students">All students</a>
    <a href="/students/new">Add student</a>
  </nav>
  <h1><%= title %></h1>
```

views/partials/footer.ejs

```html
  <p class="muted">Student Manager - built with Express, MongoDB and EJS</p>
</body>
</html>
```

![Every page is built from the same header and footer around its own content](images/21-mvc-architecture/partials.gif)

| Code                                              | Meaning                                         |
| ------------------------------------------------- | ----------------------------------------------- |
| `<%- include("../partials/header", { title }) %>` | Insert header.ejs here and give it `title`      |
| `../`                                             | The path is relative to the current view file (views/students/) |
| `<%-` (not `<%=`)                                 | The included HTML must not be escaped           |

Change the menu once in header.ejs, and every page changes.

---

## Escaping and XSS

What if someone adds a student named `<script>alert('hi')</script>`?

**XSS** (Cross-Site Scripting) is when an attacker gets their JavaScript to run in other people's browsers, for example to steal their login. It happens when user input is put into HTML without escaping.

![<%= shows the script as text, <%- runs it](images/21-mvc-architecture/xss.gif)

We tested both tags with that name

```javascript
const ejs = require("ejs");
const name = '<script>alert("hi")</script>';

console.log(ejs.render("<p><%= name %></p>", { name }));
console.log(ejs.render("<p><%- name %></p>", { name }));
```

Output

```text
<p>&lt;script&gt;alert(&#34;hi&#34;)&lt;/script&gt;</p>
<p><script>alert("hi")</script></p>
```

| Tag      | Result                                           |
| -------- | ------------------------------------------------ |
| `<%= %>` | `<` becomes `&lt;`. The browser shows the text, the script never runs |
| `<%- %>` | The real `<script>` tag. The browser runs it      |

Rule: use `<%= %>` for all data. Use `<%- %>` only for `include` and HTML you wrote yourself.

This is also why the string-building code in [Why Use MVC](#why-use-mvc) was dangerous: `` `<p>${s.name}</p>` `` does no escaping at all.

---

## Controller Layer

The Controller handles requests and coordinates Model and View. We have two controllers: one sends JSON, one renders pages. Both use the **same model**.

controllers/studentController.js (JSON API, like Session 20)

```javascript
const Student = require("../models/Student");

const ALLOWED_FIELDS = ["name", "age", "course", "email", "grade", "isActive"];

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

// GET /api/students?course=
const getAllStudents = async (req, res) => {
  const students = req.query.course
    ? await Student.findByCourse(req.query.course)
    : await Student.find().sort("name");

  res.status(200).json({ success: true, count: students.length, data: students });
};

// GET /api/students/:id
const getStudentById = async (req, res) => {
  const student = await Student.findById(req.params.id);

  if (!student) {
    return res.status(404).json({ success: false, message: "Student not found" });
  }

  res.status(200).json({ success: true, data: student, summary: student.getSummary() });
};

// POST /api/students
const createStudent = async (req, res) => {
  const student = await Student.create(pickFields(req.body || {}));

  res.status(201).json({ success: true, data: student });
};

// PATCH /api/students/:id
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

module.exports = {
  pickFields,
  getAllStudents,
  getStudentById,
  createStudent,
  updateStudent,
  deleteStudent
};
```

`condition ? a : b` (the ternary operator) is a short if/else: if a course was given, use `findByCourse`, otherwise get everyone. `pickFields` is exported too, so the page controller can reuse it.

controllers/pageController.js (HTML pages)

```javascript
const Student = require("../models/Student");
const { pickFields } = require("./studentController");

// GET /students - list page
const showStudentList = async (req, res) => {
  const students = await Student.find().sort("name");

  res.render("students/list", { title: "All Students", students });
};

// GET /students/new - empty form
const showNewForm = (req, res) => {
  res.render("students/new", { title: "Add Student", errors: [], values: {} });
};

// POST /students - the form was sent
const createFromForm = async (req, res) => {
  const values = pickFields(req.body || {});

  try {
    await Student.create(values);
  } catch (err) {
    // Show the same form again with the messages and the typed values
    if (err.name === "ValidationError" || err.code === 11000) {
      const errors = err.code === 11000
        ? ["Email already exists"]
        : Object.values(err.errors).map((e) => e.message);
      return res.status(400).render("students/new", { title: "Add Student", errors, values });
    }
    throw err; // anything else goes to the error handler
  }

  res.redirect("/students"); // Post/Redirect/Get
};

// GET /students/:id - detail page
const showStudent = async (req, res) => {
  const student = await Student.findById(req.params.id);

  if (!student) {
    return res.status(404).render("error", { title: "Not found", message: "Student not found" });
  }

  res.render("students/detail", { title: student.name, student });
};

module.exports = { showStudentList, showNewForm, createFromForm, showStudent };
```

![Two controllers, one model: JSON for apps, HTML for browsers](images/21-mvc-architecture/two-controllers.gif)

| API controller                  | Page controller                              |
| ------------------------------- | -------------------------------------------- |
| `res.json({ success, data })`   | `res.render("students/list", { students })`  |
| Errors go to the error handler as JSON | Form errors are shown on the form itself |
| After POST: 201 + the new student | After POST: redirect to the list           |

Both controllers are short. Neither builds HTML or contains schema rules.

Why does `createFromForm` have a try / catch, when Session 20 said we do not need one? Here we **want** to handle validation errors ourselves, to show them on the form. Other errors are passed on with `throw err`, so the error handler still deals with them.

---

## Forms and Post/Redirect/Get

views/students/new.ejs

```html
<%- include("../partials/header", { title }) %>

<% if (errors.length > 0) { %>
  <ul class="errors">
    <% errors.forEach((message) => { %>
      <li><%= message %></li>
    <% }); %>
  </ul>
<% } %>

<form method="POST" action="/students">
  <label>Name <input name="name" value="<%= values.name || "" %>"></label>
  <label>Age <input name="age" type="number" value="<%= values.age || "" %>"></label>
  <label>Course <input name="course" value="<%= values.course || "" %>"></label>
  <label>Email <input name="email" type="email" value="<%= values.email || "" %>"></label>
  <label>Grade
    <select name="grade">
      <% ["A", "B", "C", "D", "F"].forEach((g) => { %>
        <option <%= values.grade === g ? "selected" : "" %>><%= g %></option>
      <% }); %>
    </select>
  </label>
  <p><button type="submit">Save student</button></p>
</form>

<%- include("../partials/footer") %>
```

How a form is sent

| Part                     | Meaning                                                 |
| ------------------------ | ------------------------------------------------------- |
| `method="POST"`          | The browser sends a POST request                        |
| `action="/students"`     | ... to this URL                                         |
| `name="email"`           | The field arrives as `req.body.email`                   |
| `value="<%= values.email \|\| "" %>"` | After an error, the typed value is shown again, so the user does not retype everything |

The browser sends form data as `name=Ali+Raza&age=21&...`, not JSON. `express.urlencoded()` reads it into `req.body` (Session 14). All values arrive as strings (`age: "21"`), and Mongoose casts them (Session 19).

HTML forms can only send GET and POST. To edit or delete from a page, use POST routes like `POST /students/:id/delete`, or call the JSON API with fetch.

**Post/Redirect/Get** (PRG)

![After saving, the server redirects, so refreshing the page does not send the form again](images/21-mvc-architecture/prg.gif)

| Step | What happens                                          |
| ---- | ----------------------------------------------------- |
| 1    | Browser sends `POST /students` with the form          |
| 2    | Server saves the student and answers `302 Found, Location: /students` |
| 3    | Browser follows it with `GET /students`               |
| 4    | The list shows the new student                        |

Why not render the list directly after POST? Then the page in the browser **is** the POST answer. Pressing refresh asks "Resend the form?", and saying yes creates the student twice. After a redirect, refresh only repeats the harmless GET.

We tested the three cases

| Form sent                          | Answer                                                     |
| ---------------------------------- | ---------------------------------------------------------- |
| Valid student                      | `302 Found`, `Location: /students`                         |
| `name=A&age=15&course=&email=bad`  | `400`, the form again with 4 messages and the typed values |
| An email that already exists       | `400`, the form again with "Email already exists"          |

---

## Routes for Pages and API

routes/pageRoutes.js

```javascript
const express = require("express");
const router = express.Router();

const {
  showStudentList,
  showNewForm,
  createFromForm,
  showStudent
} = require("../controllers/pageController");

router.get("/", showStudentList);
router.get("/new", showNewForm); // before /:id
router.post("/", createFromForm);
router.get("/:id", showStudent);

module.exports = router;
```

routes/studentRoutes.js

```javascript
const express = require("express");
const router = express.Router();

const {
  getAllStudents,
  getStudentById,
  createStudent,
  updateStudent,
  deleteStudent
} = require("../controllers/studentController");

router.route("/")
  .get(getAllStudents)
  .post(createStudent);

router.route("/:id")
  .get(getStudentById)
  .patch(updateStudent)
  .delete(deleteStudent);

module.exports = router;
```

| URL prefix        | Router           | Answers with |
| ----------------- | ---------------- | ------------ |
| `/students`       | pageRoutes.js    | HTML pages   |
| `/api/students`   | studentRoutes.js | JSON         |

`/new` comes before `/:id` (Session 20), otherwise "new" would be treated as an id.

---

## Error Pages for Browsers

A browser user should see an error **page**, an app should get error **JSON**. The error handler decides with `req.accepts()`

middleware/errorHandler.js

```javascript
// Send an error as an HTML page to browsers, and as JSON to everyone else
function sendError(req, res, status, message, errors) {
  if (req.accepts(["json", "html"]) === "html") {
    return res.status(status).render("error", { title: "Error", message });
  }
  res.status(status).json({ success: false, message, errors });
}

function notFound(req, res) {
  sendError(req, res, 404, `Route ${req.method} ${req.originalUrl} not found`);
}

function errorHandler(err, req, res, next) {
  if (err.type === "entity.parse.failed") {
    return sendError(req, res, 400, "Request body is not valid JSON");
  }
  if (err.name === "ValidationError") {
    const errors = Object.values(err.errors).map((e) => e.message);
    return sendError(req, res, 400, "Validation failed", errors);
  }
  if (err.name === "CastError") {
    return sendError(req, res, 400, `Invalid ${err.path}: ${err.value}`);
  }
  if (err.code === 11000) {
    return sendError(req, res, 409, "Email already exists");
  }

  console.error(err);
  sendError(req, res, 500, "Something went wrong on the server");
}

module.exports = { notFound, errorHandler };
```

views/error.ejs

```html
<%- include("partials/header", { title }) %>

<p class="errors"><%= message %></p>

<a href="/students">Back to all students</a>

<%- include("partials/footer") %>
```

How `req.accepts()` decides

| Client                          | Sends header `Accept:`              | `req.accepts(["json", "html"])` |
| ------------------------------- | ----------------------------------- | ------------------------------- |
| Browser opening a page          | `text/html, ...`                    | `"html"`                        |
| fetch, curl, Postman (default)  | `*/*` (anything)                    | `"json"` (the first in our list) |

We tested `GET /nope`: with `Accept: text/html` we got the error page, with curl's default we got `{"success":false,"message":"Route GET /nope not found"}`. A bad id on a page (`/students/123`) shows the error page with "Invalid _id: 123" and status 400.

---

## Complete MVC Example

seed.js

```javascript
require("dotenv").config({ quiet: true });
const mongoose = require("mongoose");
const Student = require("./models/Student");

const students = [
  { name: "John Doe", age: 20, course: "Computer Science", email: "john@example.com", grade: "A" },
  { name: "Jane Smith", age: 22, course: "Mathematics", email: "jane@example.com", grade: "B" },
  { name: "Mike Johnson", age: 21, course: "Physics", email: "mike@example.com", grade: "C" },
  { name: "Sara Khan", age: 24, course: "Computer Science", email: "sara@example.com", grade: "A" }
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

server.js

```javascript
require("dotenv").config({ quiet: true });
const express = require("express");
const mongoose = require("mongoose");
const path = require("path");

const studentRoutes = require("./routes/studentRoutes");
const pageRoutes = require("./routes/pageRoutes");
const { notFound, errorHandler } = require("./middleware/errorHandler");

const app = express();
const PORT = process.env.PORT || 5000;

// View engine: res.render("students/list") uses views/students/list.ejs
app.set("view engine", "ejs");
app.set("views", path.join(__dirname, "views"));

// Middleware
app.use(express.json()); // JSON bodies (API)
app.use(express.urlencoded()); // HTML form bodies (pages)

// Routes
app.get("/", (req, res) => res.redirect("/students"));
app.use("/students", pageRoutes); // HTML pages
app.use("/api/students", studentRoutes); // JSON API

// 404 and errors (always last)
app.use(notFound);
app.use(errorHandler);

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
    console.log(`Pages: http://localhost:${PORT}/students`);
    console.log(`API:   http://localhost:${PORT}/api/students`);
  });
}

startServer();
```

Run it

```bash
node seed.js
node --watch server.js
```

Output

```text
Added students: 4
Connected to MongoDB
Server running on port 5000
Pages: http://localhost:5000/students
API:   http://localhost:5000/api/students
```

Try it in the browser

| Open                                         | You see                                     |
| -------------------------------------------- | ------------------------------------------- |
| `http://localhost:5000/`                     | Redirected to the student list              |
| `http://localhost:5000/students`             | 4 student cards, sorted by name             |
| Click a name                                 | The detail page with the summary            |
| `http://localhost:5000/students/new`         | The form. Send it empty to see the messages |
| `http://localhost:5000/api/students?course=Computer%20Science` | JSON with John Doe and Sara Khan |
| `http://localhost:5000/nope`                 | The error page                              |

Tested with a JSON client

```text
GET /api/students?course=Computer Science   → 2 students: John Doe, Sara Khan
GET /api/students/:id                        → includes "summary": "Ali Raza is studying Physics and got grade A"
POST /api/students with invalid data         → 400 ["Name must be at least 2 characters","Age must be at least 18","Please enter a valid email","Q is not a valid grade"]
POST /api/students with broken JSON          → 400 Request body is not valid JSON
```

---

## How MVC Works Together

Flow of a request to `GET /students`

![The full journey of one page request through the MVC layers](images/21-mvc-architecture/mvc-flow.gif)

| Step | Where                     | What happens                                      |
| ---- | ------------------------- | ------------------------------------------------- |
| 1    | Browser                   | Sends `GET /students`                             |
| 2    | server.js                 | `app.use("/students", pageRoutes)`                |
| 3    | routes/pageRoutes.js      | `/` → `showStudentList`                           |
| 4    | Controller                | Calls `Student.find().sort("name")`               |
| 5    | Model                     | Mongoose gets the students from MongoDB           |
| 6    | Controller                | `res.render("students/list", { students })`       |
| 7    | View                      | EJS fills list.ejs with the students              |
| 8    | Browser                   | Receives finished HTML and shows it               |

Each layer has one job

| Layer      | Job               |
| ---------- | ----------------- |
| Model      | Get and check data |
| View       | Show data         |
| Controller | Connect the two   |

In a React or mobile app, steps 6 to 8 happen differently: the server sends JSON (the API controller), and the app builds the screen itself. The model and the MVC idea stay the same.

---

## Beginner Mistakes

### Mistake 1

Forgetting `app.set("view engine", "ejs")`.

`res.render("students/list")` fails with "No default engine was specified and no extension was provided".

---

### Mistake 2

Wrong path in `res.render()`.

The path is relative to the views folder: `res.render("students/list")`, not `"views/students/list"` or `"./list"`.

---

### Mistake 3

Using `<%-` for user data.

`<%- student.name %>` lets a name with `<script>` run in every visitor's browser (XSS). Use `<%=`.

---

### Mistake 4

Forgetting a variable when rendering.

If list.ejs uses `title` but the controller only passes `{ students }`, the page fails with "title is not defined". Pass every variable the view uses.

---

### Mistake 5

Rendering a page after a form POST instead of redirecting.

Refresh sends the form again and creates duplicates. Use `res.redirect()` after a successful POST.

---

### Mistake 6

Forgetting `express.urlencoded()`.

Form fields arrive in a format `express.json()` cannot read, so `req.body` is `undefined`.

---

### Mistake 7

Database queries inside a view.

A view gets data from the controller. It never calls `Student.find()`.

---

### Mistake 8

Arrow functions for methods and statics.

`this` would not be the student or the model (Session 19).

---

## Practice Exercises

### Exercise 1

Create a Course model with name, description, duration and price. Add a static `findCheaperThan(price)`

### Exercise 2

Create a courseController.js for the JSON API with getAllCourses, getCourseById, createCourse, updateCourse and deleteCourse

### Exercise 3

Create views/courses/list.ejs that shows every course in a card, using the header and footer partials

### Exercise 4

Add a "Delete" button on the student detail page. Use a form with `method="POST"` and `action="/students/<%= student._id %>/delete"`, and a route that deletes the student and redirects to the list

### Exercise 5

Create a dashboard page at `/dashboard` that shows

* Total students
* Active students
* Number of students per grade
* Average student age (add all ages, divide by the count)

Put the counting in a static method `Student.getDashboardStats()`

### Exercise 6

Add an "Edit" page with the form filled in. Send it with POST to `/students/:id/edit`, show errors on the form, and redirect to the detail page on success

### Exercise 7

Add a student named `<b>Bold</b>` through the form. Check that the list shows the tags as text. Change `<%=` to `<%-` in list.ejs and look again. Change it back

---

## Interview Questions

### What does MVC stand for

Model View Controller

### What is the role of Model in MVC

Model handles data, database operations and data rules like validation

### What is the role of View in MVC

View handles what the user sees: HTML pages, or the JSON shape in an API

### What is the role of Controller in MVC

Controller handles requests, calls the model, and chooses which view or response to send

### Where do routes fit in MVC

Routes are not a letter of MVC. They map each URL and method to the right controller

### Why do we use MVC pattern

To separate concerns, organize code, reuse models, and make applications easier to maintain and to work on in teams

### What does "fat model, thin controller" mean

Put data logic (validation, reusable queries, computed values) in the model, and keep controllers short

### What is the difference between an instance method and a static method

An instance method runs on one document (this = the document). A static runs on the model (this = the model), for queries over the collection

### What is a template engine

A tool that fills an HTML template with data to produce a page. EJS is one

### What is the difference between <%= and <%- in EJS

<%= escapes HTML (safe for user data). <%- prints raw HTML (only for include and trusted HTML)

### What is XSS

Cross-Site Scripting. An attacker's JavaScript runs in other users' browsers because user input was put into HTML without escaping

### What are partials

Small templates, like a header and footer, included in many views so they are written only once

### What is Post/Redirect/Get

After a successful form POST, the server redirects to a GET page. Refreshing then does not send the form again

### Can MVC be used for both APIs and web applications

Yes
The same model can serve an HTML page controller and a JSON API controller

---
