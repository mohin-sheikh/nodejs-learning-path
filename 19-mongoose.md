## Table of Contents

* [What is Mongoose](#what-is-mongoose)
* [Why Use Mongoose](#why-use-mongoose)
* [Installing Mongoose](#installing-mongoose)
* [Connecting to MongoDB with Mongoose](#connecting-to-mongodb-with-mongoose)
* [What is a Schema](#what-is-a-schema)
* [Creating a Schema](#creating-a-schema)
* [What is a Model](#what-is-a-model)
* [Creating a Model](#creating-a-model)
* [What a Saved Document Looks Like](#what-a-saved-document-looks-like)
* [CRUD Operations with Mongoose](#crud-operations-with-mongoose)
* [Create Operation](#create-operation)
* [Read Operation](#read-operation)
* [Update Operation](#update-operation)
* [Delete Operation](#delete-operation)
* [Schema Validation](#schema-validation)
* [Handling Mongoose Errors](#handling-mongoose-errors)
* [Middleware Hooks](#middleware-hooks)
* [Instance Methods](#instance-methods)
* [Hiding Fields with select false](#hiding-fields-with-select-false)
* [Relationships with ref and populate](#relationships-with-ref-and-populate)
* [Complete Student API with Mongoose](#complete-student-api-with-mongoose)
* [Testing the API](#testing-the-api)
* [Beginner Mistakes](#beginner-mistakes)
* [Practice Exercises](#practice-exercises)
* [Interview Questions](#interview-questions)

---

## What is Mongoose

Mongoose is an ODM for MongoDB

ODM stands for Object Data Modeling

It helps you work with MongoDB using JavaScript objects, with rules about what those objects must look like

Think of it like a translator with a checklist

![Mongoose checks your object against the schema, then talks to MongoDB](images/19-mongoose/what-is-mongoose.gif)

* Your code gives Mongoose a JavaScript object
* Mongoose checks it against the rules (the schema) and fixes types
* Mongoose sends it to MongoDB using the same driver you used in Session 18
* Results come back as Mongoose documents, objects with extra methods like `save()`

Without Mongoose, we wrote code like this (Session 18)

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

In Session 18 we wrote a lot of code by hand. Mongoose does much of it for us

| Task                         | MongoDB driver (Session 18)                    | Mongoose (this session)                    |
| ---------------------------- | ---------------------------------------------- | ------------------------------------------ |
| Structure of a student       | Nothing. Any object is saved                   | A schema lists the fields and types        |
| Validation                   | Our own `validateStudent()` function           | Rules in the schema, checked on every save |
| Unknown fields like `role`   | Saved, unless we pick fields ourselves         | Dropped automatically                      |
| Default values               | `course: course \|\| "Not specified"`          | `default: "Not specified"`                 |
| createdAt / updatedAt        | Set by hand with `new Date()`                  | `timestamps: true`                         |
| Find by id                   | `ObjectId.isValid()` + `new ObjectId(id)`      | `findById(id)`                             |
| Invalid id                   | `checkId` middleware                           | A `CastError`, handled in one place        |
| Unique email                 | `createIndex()` by hand                        | `unique: true` in the schema               |
| Related data                 | Two queries by hand                            | `populate()`                               |

![The same task in the driver and in Mongoose](images/19-mongoose/driver-vs-mongoose.gif)

Mongoose is not a different database. It is a package that sits on top of the official MongoDB driver. Everything you learned in Sessions 17 and 18 (filters, `$set`, `$gt`, ObjectId, indexes) still applies.

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

You do not need `npm install mongodb`. Mongoose installs the driver for you.

Create a .env file

```text
MONGODB_URI=mongodb+srv://yourusername:yourpassword@cluster0.abc123.mongodb.net/
DB_NAME=school
PORT=3000
```

For a local MongoDB, use `MONGODB_URI=mongodb://127.0.0.1:27017` (Session 17)

This session uses Mongoose 9. Some older tutorials show code that no longer works in Mongoose 9. Those places are marked in this session.

---

## Connecting to MongoDB with Mongoose

Connecting with Mongoose is one line

```javascript
require("dotenv").config({ quiet: true });
const mongoose = require("mongoose");

async function main() {
  await mongoose.connect(process.env.MONGODB_URI, { dbName: process.env.DB_NAME });
  console.log("Connected to MongoDB");

  await mongoose.disconnect();
}

main();
```

Output

```text
Connected to MongoDB
```

`mongoose.disconnect()` closes the connection so the script can end, like `client.close()` in Session 17. A server does not disconnect, it keeps the connection open.

`{ dbName: ... }` chooses the database. Do **not** glue the database name onto the connection string

```javascript
// Do not do this
mongoose.connect(`${process.env.MONGODB_URI}/${process.env.DB_NAME}`);
```

We tested what goes wrong

| MONGODB_URI ends with           | Glued string                          | Database actually used          |
| ------------------------------- | ------------------------------------- | ------------------------------- |
| `.../`                          | `...27017//school`                    | `/school` (a strange name with a slash) |
| `.../?retryWrites=true` (Atlas) | `.../?retryWrites=true/school`        | `test`, your data goes to the wrong database |

No error is shown in either case. Your data is simply somewhere else. The `dbName` option always works.

Like in Session 18, connect once and start the server only after the connection is ready

```javascript
async function startServer() {
  try {
    await mongoose.connect(process.env.MONGODB_URI, { dbName: process.env.DB_NAME });
    console.log("Connected to MongoDB");
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

You no longer need db.js with `getDB()`. Mongoose keeps the connection inside the `mongoose` object, and every model uses it automatically. Any file that does `require("mongoose")` gets the same connected object (module caching, Session 04).

What if you forget to connect? Mongoose does not fail right away. It **buffers** (waits with) the query for 10 seconds, hoping a connection will come, then throws

```text
MongooseError: Operation `students.find()` buffering timed out after 10000ms
```

If a route takes exactly 10 seconds and then fails, check that `mongoose.connect()` is really called and awaited.

Connection events you can listen to (Session 08)

```javascript
mongoose.connection.on("connected", () => {
  console.log("Mongoose connected to MongoDB");
});

mongoose.connection.on("error", (err) => {
  console.log("Mongoose connection error", err.message);
});

mongoose.connection.on("disconnected", () => {
  console.log("Mongoose disconnected");
});
```

`mongoose.connection` is an EventEmitter, just like the ones you built in Session 08. `disconnected` is useful on a server: it tells you when the database connection is lost (for example, Atlas restarted).

---

## What is a Schema

A schema defines the structure of your documents

It tells Mongoose what fields a document should have

Think of a schema like a blueprint

![A schema is a blueprint: every document is built from it](images/19-mongoose/schema-blueprint.gif)

| A house blueprint tells you    | A schema tells you                   |
| ------------------------------ | ------------------------------------ |
| What rooms a house has         | What fields a document has           |
| What size each room is         | What type each field is              |
| What rules the builder follows | What validation is required          |

Example of a student schema

```javascript
const studentSchema = new mongoose.Schema({
  name: String,
  age: Number,
  course: String,
  email: String
});
```

This means a student document can have these four fields, with these types. Any other field is ignored when you save.

Note: MongoDB itself still has no fixed structure (Session 17). The schema lives in your Node.js code. If another program writes to the same collection without Mongoose, those rules do not apply.

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
  }
});
```

Schema field options

| Option      | Works on       | Meaning                                         |
| ----------- | -------------- | ----------------------------------------------- |
| `type`      | all            | Data type (String, Number, Date, Boolean, Array, ObjectId) |
| `required`  | all            | Field must be provided                          |
| `default`   | all            | Value used if the field is not provided         |
| `unique`    | all            | Creates a unique index (Session 18). Not a validator |
| `min` / `max` | Number, Date | Smallest / largest allowed value                |
| `minlength` / `maxlength` | String | Shortest / longest allowed text          |
| `enum`      | String         | Value must be one of a list                     |
| `match`     | String         | Value must match a pattern                      |
| `lowercase` | String         | Convert to lowercase before saving              |
| `uppercase` | String         | Convert to uppercase before saving              |
| `trim`      | String         | Remove spaces at both ends before saving        |
| `select`    | all            | `false` hides the field from query results      |

Every rule can have its own message: write `[value, "message"]` instead of just the value

```javascript
age: {
  type: Number,
  required: [true, "Age is required"],
  min: [18, "Age must be at least 18"]
}
```

Without a message, Mongoose writes its own, like ``Path `age` is required.``

Schema options (the second argument)

```javascript
const studentSchema = new mongoose.Schema(
  { name: String },
  { timestamps: true }
);
```

`timestamps: true` adds `createdAt` and `updatedAt`, and keeps `updatedAt` correct on every save and update.

---

## What is a Model

A model is a wrapper around a schema

It gives you methods to work with a specific collection

![The schema is the blueprint, the model is the factory that makes documents](images/19-mongoose/schema-model-document.gif)

| Thing     | Like a        | Example                                     |
| --------- | ------------- | ------------------------------------------- |
| Schema    | Blueprint     | `studentSchema`                             |
| Model     | Factory       | `Student`, has `find()`, `create()`, ...    |
| Document  | One product   | `john`, one student, has `save()`           |

The model name should be singular and start with a capital letter

```javascript
const Student = mongoose.model("Student", studentSchema);
```

Mongoose turns the model name into the collection name: lowercase and plural

| Model name  | Collection   |
| ----------- | ------------ |
| `Student`   | `students`   |
| `Category`  | `categories` |
| `Person`    | `people`     |

Mongoose knows English plurals, so `Person` becomes `people`. If you see a collection you did not expect in Compass, this is why.

Now you can use Student to perform CRUD operations

---

## Creating a Model

Put each model in its own file inside a models folder

Create a file models/Student.js

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
      trim: true,
      default: "Not specified"
    },
    email: {
      type: String,
      required: [true, "Email is required"],
      unique: true,
      lowercase: true,
      trim: true,
      match: [/^\S+@\S+\.\S+$/, "Please enter a valid email"]
    }
  },
  {
    timestamps: true // Adds createdAt and updatedAt automatically
  }
);

module.exports = mongoose.model("Student", studentSchema);
```

New options here

| Option                            | Meaning                                                    |
| --------------------------------- | ---------------------------------------------------------- |
| `cast: "Age must be a number"`    | Message when the value cannot become a number, like `"abc"` |
| `match: [/^\S+@\S+\.\S+$/, ...]`  | The email must look like `text@text.text`                  |

`/^\S+@\S+\.\S+$/` is a **regular expression**, a pattern for text. `\S+` means "one or more characters that are not spaces", `^` means start and `$` means end. You do not need to write your own patterns yet. Copy this one.

Now you can import and use this model anywhere

```javascript
const Student = require("./models/Student");
```

---

## What a Saved Document Looks Like

Save a student with messy input

```javascript
const student = new Student({
  name: "  John Doe  ",
  age: "20",
  email: "  JOHN@Example.com ",
  role: "admin"
});

await student.save();
console.log(student);
```

Output

```text
{
  name: 'John Doe',
  age: 20,
  course: 'Not specified',
  email: 'john@example.com',
  _id: new ObjectId('6ac20a45fbe441e1332b13e4'),
  createdAt: 2026-10-04T08:11:49.838Z,
  updatedAt: 2026-10-04T08:11:49.838Z,
  __v: 0
}
```

![Mongoose cleans the input: trim, lowercase, cast, default, and drops unknown fields](images/19-mongoose/clean-input.gif)

| Input                      | Saved                  | Why                                       |
| -------------------------- | ---------------------- | ----------------------------------------- |
| `"  John Doe  "`           | `"John Doe"`           | `trim: true`                              |
| `"20"` (a string)          | `20` (a number)        | Mongoose **casts** to the schema type     |
| no course                  | `"Not specified"`      | `default`                                 |
| `"  JOHN@Example.com "`    | `"john@example.com"`   | `trim` + `lowercase`                      |
| `role: "admin"`            | not saved              | Not in the schema, so it is dropped       |
| -                          | `_id`                  | Created by Mongoose before saving         |
| -                          | `createdAt`, `updatedAt` | `timestamps: true`                      |
| -                          | `__v: 0`               | Version key (see below)                   |

**Casting** is a big difference from Session 15 and 18. Our `validateStudent()` rejected `"20"` because `typeof "20"` is `"string"`. Mongoose converts `"20"` to `20` because the schema says Number. `"abc"` cannot be converted, so it fails with "Age must be a number".

**`__v`** is the version key. Mongoose adds it to every document and uses it to protect arrays from conflicting updates. You can ignore it, or hide it with `.select("-__v")`.

**`id` and `_id`**: every document has `_id` (an ObjectId) and a shortcut `student.id`, which is the same value as a string. `res.json()` sends `_id` as a string.

---

## CRUD Operations with Mongoose

Mongoose makes CRUD operations very simple. All methods are called on the model (`Student`), except `save()`, which is called on one document

| Operation | Methods                                                       |
| --------- | ------------------------------------------------------------- |
| Create    | `new Student()` + `save()`, `create()`, `insertMany()`        |
| Read      | `find()`, `findOne()`, `findById()`, `countDocuments()`, `exists()` |
| Update    | `findByIdAndUpdate()`, `updateOne()`, `updateMany()`, find + `save()` |
| Delete    | `findByIdAndDelete()`, `deleteOne()`, `deleteMany()`          |

All of them return something you `await`

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
console.log(savedStudent === student);
```

Output

```text
true
```

`save()` returns the same document, now with `_id`, `createdAt` and `updatedAt`.

Method 2 - Create directly

```javascript
const student = await Student.create({
  name: "John Doe",
  age: 20,
  course: "Computer Science",
  email: "john@example.com"
});
```

`create()` is `new Student()` + `save()` in one step. This is what most APIs use.

Method 3 - Create multiple

```javascript
const students = await Student.insertMany([
  { name: "John Doe", age: 20, email: "john@example.com" },
  { name: "Jane Smith", age: 22, email: "jane@example.com" }
]);

console.log(students.length);
```

Output

```text
2
```

Every document is validated before anything is saved.

---

## Read Operation

Find all documents

```javascript
const allStudents = await Student.find();
```

With Mongoose there is no `.toArray()`. `await` gives you the array directly.

Find with filter (same filters as Session 18)

```javascript
const csStudents = await Student.find({ course: "Computer Science" });
const older = await Student.find({ age: { $gte: 21 } });
```

Find one document

```javascript
const student = await Student.findOne({ email: "john@example.com" });
```

Find by id

```javascript
const student = await Student.findById(req.params.id);
```

No `new ObjectId()` needed. Mongoose converts the string for you. If nothing has that id, the result is `null`. If the id is not a valid ObjectId (like `"123"`), it throws a `CastError` (see [Handling Mongoose Errors](#handling-mongoose-errors)).

Select specific fields (a projection, Session 18)

```javascript
// Only name and email (and _id)
const students = await Student.find().select("name email");

// Everything except __v
const students = await Student.find().select("-__v");
```

Sort, limit and skip

```javascript
// Ascending by name
const students = await Student.find().sort("name");

// Descending by age (minus sign)
const students = await Student.find().sort("-age");

// Page 3 with 10 per page
const students = await Student.find().sort("name").skip(20).limit(10);
```

| Mongoose          | Driver (Session 18)       |
| ----------------- | ------------------------- |
| `.sort("age")`    | `.sort({ age: 1 })`       |
| `.sort("-age")`   | `.sort({ age: -1 })`      |
| `.select("name email")` | `{ projection: { name: 1, email: 1 } }` |

Mongoose also accepts the object form, like `.sort({ age: -1 })`.

Count and check

```javascript
const count = await Student.countDocuments({ course: "Computer Science" });

const found = await Student.exists({ email: "john@example.com" });
```

`exists()` returns `{ _id: ... }` if a document matches, or `null`. It is faster than `findOne()` when you only need yes or no.

![A query is built step by step and runs when you await it](images/19-mongoose/query-chain.gif)

A query does not run when you write it. `Student.find()` builds a **Query** object. `.sort()`, `.select()`, `.limit()` add to it. The database is asked only when you `await` it.

Plain objects with lean()

```javascript
const students = await Student.find().lean();
```

Normal results are Mongoose documents with methods like `save()`. `.lean()` returns plain JavaScript objects instead, which is faster for read-only routes. You cannot call `save()` on them.

---

## Update Operation

There are two ways to update

![findByIdAndUpdate changes the database directly, find + save goes through the document](images/19-mongoose/update-two-ways.gif)

Way 1 - findByIdAndUpdate (one database call)

```javascript
const student = await Student.findByIdAndUpdate(
  req.params.id,
  { age: 21 },
  { returnDocument: "after", runValidators: true }
);
```

| Option                       | Meaning                                                      |
| ---------------------------- | ------------------------------------------------------------ |
| `{ age: 21 }`                | Mongoose turns this into `{ $set: { age: 21 } }` for you     |
| `returnDocument: "after"`    | Return the student after the change. Default is the old one  |
| `runValidators: true`        | Check the schema rules. **Updates skip validation by default** |

The result is `null` if no student has that id.

`runValidators` is easy to forget. We tested it: without it, `{ age: 5 }` is saved although the schema says `min: 18`.

Older tutorials write `{ new: true }`. In Mongoose 9 it still works but prints a warning

```text
[MONGOOSE] Warning: mongoose: the `new` option for `findOneAndUpdate()` and `findOneAndReplace()` is deprecated. Use `returnDocument: 'after'` instead.
```

Way 2 - Find, change, save

```javascript
const student = await Student.findById(req.params.id);

if (!student) {
  return res.status(404).json({ success: false, message: "Student not found" });
}

student.age = 21;
await student.save();
```

| Way                      | Database calls | Validation          | Runs save hooks |
| ------------------------ | -------------- | ------------------- | --------------- |
| `findByIdAndUpdate`      | 1              | Only with `runValidators: true` | No  |
| find + `save()`          | 2              | Always              | Yes             |

You will use find + `save()` in Session 23, because password hashing runs in a save hook.

Update operators from Session 18 still work

```javascript
await Student.findByIdAndUpdate(id, { $inc: { age: 1 } });
```

Update many documents

```javascript
const result = await Student.updateMany(
  { course: "Mathematics" },
  { course: "Maths" }
);

console.log(result.matchedCount, result.modifiedCount);
```

---

## Delete Operation

Delete by id

```javascript
const student = await Student.findByIdAndDelete(req.params.id);
```

Returns the deleted student, or `null` if none had that id. Perfect for a 404 check.

Delete one document by a filter

```javascript
const result = await Student.deleteOne({ email: "john@example.com" });
console.log(result.deletedCount);
```

Delete multiple documents

```javascript
const result = await Student.deleteMany({ age: { $lt: 18 } });
```

Delete all documents

```javascript
const result = await Student.deleteMany({});
```

Be careful: there is no undo.

---

## Schema Validation

Mongoose validates data before saving

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
      message: "{VALUE} is not a valid course"
    }
  }
});
```

`{VALUE}` in a message is replaced with the value that failed. Sending `"Art"` gives "Art is not a valid course".

Custom validator

```javascript
const contactSchema = new mongoose.Schema({
  phoneNumber: {
    type: String,
    validate: {
      validator: function (value) {
        return /^\d{10}$/.test(value);
      },
      message: "Phone number must be exactly 10 digits"
    }
  }
});
```

The validator function returns true (valid) or false (invalid). `\d{10}` means exactly 10 digits, and `^ ... $` makes sure there is nothing before or after them. Without `^` and `$`, `"12345678901"` (11 digits) would pass, because it contains 10 digits.

When does validation run?

| Code                                         | Validation runs?             |
| -------------------------------------------- | ---------------------------- |
| `create()`, `save()`, `insertMany()`         | Yes                          |
| `findByIdAndUpdate()`, `updateOne()`, `updateMany()` | Only with `runValidators: true` |

`unique: true` is not a validator. It creates a unique index, and MongoDB itself rejects duplicates with error E11000 (Session 18). Mongoose builds the index when the app starts. If the collection already has duplicates, the index cannot be built, and duplicates are not blocked. Clean the data first.

---

## Handling Mongoose Errors

Mongoose throws errors with a `name` you can check. These are the ones you will meet every day

![Each Mongoose error becomes the right status code](images/19-mongoose/error-types.gif)

**ValidationError** - schema rules failed

```javascript
try {
  await Student.create({ name: "J", age: "abc", email: "bad" });
} catch (err) {
  console.log(err.name);
  console.log(Object.keys(err.errors));
  console.log(Object.values(err.errors).map((e) => e.message));
}
```

Output

```text
ValidationError
[ 'age', 'name', 'email' ]
[
  'Age must be a number',
  'Name must be at least 2 characters',
  'Please enter a valid email'
]
```

`err.errors` is an object with one entry per field that failed. `Object.values()` gives the entries as an array, and `map()` takes the message from each. Sessions 24, 29 and 30 use exactly this line.

**CastError** - a value has the wrong type, most often a bad id

```javascript
await Student.findById("123");
```

```text
CastError: Cast to ObjectId failed for value "123" (type string) at path "_id" for model "Student"
```

The error has `err.path` (`"_id"`) and `err.value` (`"123"`). This replaces the `checkId` middleware from Session 18.

You can also check an id yourself with `mongoose.isValidObjectId("123")`, which returns `false`.

**Duplicate key (code 11000)** - the unique index blocked a duplicate

```text
MongoServerError: E11000 duplicate key error collection: school.students index: email_1 dup key: { email: "john@example.com" }
```

This one comes from MongoDB, not Mongoose, so check `err.code === 11000`, not the name.

| Error                 | Check                           | Status |
| --------------------- | ------------------------------- | ------ |
| ValidationError       | `err.name === "ValidationError"` | 400   |
| CastError             | `err.name === "CastError"`       | 400   |
| Duplicate key         | `err.code === 11000`             | 409   |
| Anything else         | -                                | 500   |

The [Complete Student API](#complete-student-api-with-mongoose) handles all of them in one error handler.

---

## Middleware Hooks

Mongoose middleware (also called hooks) are functions that run automatically before or after an operation. They work like Express middleware, but for the database.

```javascript
const productSchema = new mongoose.Schema({
  name: { type: String, required: true },
  slug: String,
  price: Number
});

// Runs before every save()
productSchema.pre("save", function () {
  if (this.isModified("name")) {
    this.slug = this.name.toLowerCase().split(" ").join("-");
  }
});
```

```javascript
const product = await Product.create({ name: "Wireless Mouse", price: 25 });
console.log(product.slug);

product.name = "Gaming Mouse";
await product.save();
console.log(product.slug);
```

Output

```text
wireless-mouse
gaming-mouse
```

![The pre save hook runs between save() and the database](images/19-mongoose/pre-save-hook.gif)

| Code                     | Meaning                                                    |
| ------------------------ | ---------------------------------------------------------- |
| `pre("save", ...)`       | Run before the document is saved                           |
| `function () { }`        | Must be a normal function, not an arrow function           |
| `this`                   | The document being saved                                   |
| `this.isModified("name")` | True if name is new or changed since it was loaded        |

Why not an arrow function? An arrow function does not get its own `this`, so `this.name` would be undefined. Mongoose sets `this` to the document only for normal functions.

Why `isModified`? Without it, the slug would be rebuilt on every save, even when only the price changed. In Session 23 you will use the same shape to hash a password only when the password changed.

**Mongoose 9 note:** older tutorials write hooks with a `next` parameter

```javascript
// Old style: breaks in Mongoose 9
schema.pre("save", async function (next) {
  if (!this.isModified("password")) return next();
  // ...
  next();
});
```

In Mongoose 9 this throws `TypeError: next is not a function`. Write hooks without `next`: use `return` to stop early, and an `async function` if you need `await`. To stop the save with an error, `throw new Error("...")`.

Hooks run only on `save()` and `create()`. We tested it: `findByIdAndUpdate(id, { name: "Office Mouse" })` changed the name, but the slug stayed `gaming-mouse`. If a hook must run, use find + `save()`.

---

## Instance Methods

You can add your own functions to every document of a model

```javascript
productSchema.methods.getLabel = function () {
  return `${this.name} costs $${this.price}`;
};
```

```javascript
const product = await Product.create({ name: "Wireless Mouse", price: 25 });
console.log(product.getLabel());
```

Output

```text
Wireless Mouse costs $25
```

Again a normal function, so `this` is the document. Methods keep logic that belongs to the data inside the model. Session 21 uses this for `getSummary()`, and Session 23 uses it for `comparePassword()`.

Add methods and hooks **before** calling `mongoose.model()`.

---

## Hiding Fields with select false

Some fields should never be sent to the client, like a password. `select: false` hides a field from every query unless you ask for it

```javascript
const userSchema = new mongoose.Schema({
  email: String,
  password: { type: String, select: false }
});
```

```javascript
const found = await User.findById(user._id);
console.log(found.password);

const withPassword = await User.findById(user._id).select("+password");
console.log(withPassword.password);
```

Output

```text
undefined
secret123
```

`.select("+password")` means "also include password". You only need it in one place: the login route, to check the password (Session 22). Everywhere else, the password is hidden automatically.

Note: `create()` returns the document you just made, which still has the password in memory. Do not send that directly in a response.

---

## Relationships with ref and populate

A course has a teacher. Instead of copying the teacher into every course, store the teacher's `_id` and link it with `ref`

```javascript
const teacherSchema = new mongoose.Schema({ name: String });
const Teacher = mongoose.model("Teacher", teacherSchema);

const courseSchema = new mongoose.Schema({
  title: String,
  teacher: { type: mongoose.Schema.Types.ObjectId, ref: "Teacher" }
});
const Course = mongoose.model("Course", courseSchema);
```

```javascript
const teacher = await Teacher.create({ name: "Ms. Lee" });
await Course.create({ title: "Node.js Basics", teacher: teacher._id });

const plain = await Course.findOne({ title: "Node.js Basics" });
console.log(plain.teacher);

const full = await Course.findOne({ title: "Node.js Basics" }).populate("teacher");
console.log(full.teacher.name);
```

Output

```text
new ObjectId('6ac20aa3f9057f3ecc3f229f')
Ms. Lee
```

![populate replaces the stored id with the real teacher document](images/19-mongoose/populate.gif)

| Part                                   | Meaning                                         |
| -------------------------------------- | ----------------------------------------------- |
| `mongoose.Schema.Types.ObjectId`       | The field stores an ObjectId                    |
| `ref: "Teacher"`                       | That id belongs to the Teacher model            |
| `.populate("teacher")`                 | Replace the id with the teacher document        |
| `.populate("teacher", "name")`         | Same, but only the teacher's name               |

If the teacher changes her name, every course shows the new name, because only the id is stored. Session 30 links tasks to users this way.

---

## Complete Student API with Mongoose

This is the same API as Session 18, rebuilt with Mongoose. Compare the length: no `validateStudent()`, no `checkId`, no `new ObjectId()`.

Project structure

```text
mongoose-api/
├── models/
│   └── Student.js
├── .env
├── .gitignore
├── seed.js
├── server.js
├── test-api.js
└── package.json
```

models/Student.js is shown in [Creating a Model](#creating-a-model).

seed.js

```javascript
require("dotenv").config({ quiet: true });
const mongoose = require("mongoose");
const Student = require("./models/Student");

const students = [
  { name: "John Doe", age: 20, course: "Computer Science", email: "john@example.com" },
  { name: "Jane Smith", age: 22, course: "Mathematics", email: "jane@example.com" },
  { name: "Mike Johnson", age: 21, course: "Physics", email: "mike@example.com" },
  { name: "Sara Khan", age: 24, course: "Computer Science", email: "sara@example.com" },
  { name: "Tom Brown", age: 19, course: "Mathematics", email: "tom@example.com" }
];

async function seed() {
  try {
    await mongoose.connect(process.env.MONGODB_URI, { dbName: process.env.DB_NAME });

    const deleted = await Student.deleteMany({});
    console.log("Removed old students:", deleted.deletedCount);

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

Output

```text
Removed old students: 0
Added students: 5
```

`mongoose.disconnect()` closes the connection so the script can end (like `client.close()` in Session 17).

server.js

```javascript
require("dotenv").config({ quiet: true });
const express = require("express");
const mongoose = require("mongoose");
const Student = require("./models/Student");

const app = express();
const PORT = process.env.PORT || 3000;

app.use(express.json());

// CREATE - POST /students
app.post("/students", async (req, res) => {
  const { name, age, course, email } = req.body || {};

  // The schema validates. If something is wrong, the error handler answers 400
  const student = await Student.create({ name, age, course, email });

  res.status(201).json({ success: true, data: student });
});

// READ ALL - GET /students?course=...&sort=...
app.get("/students", async (req, res) => {
  const filter = {};

  if (req.query.course) {
    filter.course = req.query.course;
  }

  const sort = req.query.sort || "createdAt";
  if (!["name", "-name", "age", "-age", "createdAt", "-createdAt"].includes(sort)) {
    return res.status(400).json({ success: false, message: "You can sort by name, age or createdAt" });
  }

  const students = await Student.find(filter).sort(sort);

  res.status(200).json({ success: true, count: students.length, data: students });
});

// READ ONE - GET /students/:id
app.get("/students/:id", async (req, res) => {
  const student = await Student.findById(req.params.id);

  if (!student) {
    return res.status(404).json({ success: false, message: "Student not found" });
  }

  res.status(200).json({ success: true, data: student });
});

// REPLACE - PUT /students/:id (every field is required)
app.put("/students/:id", async (req, res) => {
  const { name, age, course, email } = req.body || {};

  const student = await Student.findById(req.params.id);

  if (!student) {
    return res.status(404).json({ success: false, message: "Student not found" });
  }

  student.set({ name, age, course: course || "Not specified", email });
  await student.save(); // save() always runs the schema validation

  res.status(200).json({ success: true, data: student });
});

// UPDATE SOME FIELDS - PATCH /students/:id
app.patch("/students/:id", async (req, res) => {
  const body = req.body || {};

  const changes = {};
  for (const key of ["name", "age", "course", "email"]) {
    if (body[key] !== undefined) {
      changes[key] = body[key];
    }
  }

  const student = await Student.findByIdAndUpdate(req.params.id, changes, {
    returnDocument: "after", // send back the student after the change
    runValidators: true // updates skip validation unless you ask for it
  });

  if (!student) {
    return res.status(404).json({ success: false, message: "Student not found" });
  }

  res.status(200).json({ success: true, data: student });
});

// DELETE - DELETE /students/:id
app.delete("/students/:id", async (req, res) => {
  const student = await Student.findByIdAndDelete(req.params.id);

  if (!student) {
    return res.status(404).json({ success: false, message: "Student not found" });
  }

  res.status(200).json({ success: true, message: "Student deleted successfully" });
});

// 404 for unknown routes
app.use((req, res) => {
  res.status(404).json({ success: false, message: `Route ${req.method} ${req.url} not found` });
});

// Error handler
app.use((err, req, res, next) => {
  // The body was not valid JSON
  if (err.type === "entity.parse.failed") {
    return res.status(400).json({ success: false, message: "Request body is not valid JSON" });
  }

  // Schema rules failed (required, min, max, match ...)
  if (err.name === "ValidationError") {
    const errors = Object.values(err.errors).map((e) => e.message);
    return res.status(400).json({ success: false, message: "Validation failed", errors });
  }

  // A value has the wrong type, for example the id "123"
  if (err.name === "CastError") {
    return res.status(400).json({ success: false, message: `Invalid ${err.path}: ${err.value}` });
  }

  // Unique index: the email already exists
  if (err.code === 11000) {
    return res.status(409).json({ success: false, message: "Email already exists" });
  }

  console.error(err);
  res.status(500).json({ success: false, message: "Something went wrong on the server" });
});

// Connect first, then start the server
async function startServer() {
  try {
    await mongoose.connect(process.env.MONGODB_URI, { dbName: process.env.DB_NAME });
    console.log("Connected to MongoDB");
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

Things to notice

| Code                                         | Why                                                      |
| -------------------------------------------- | -------------------------------------------------------- |
| `Student.create({ name, age, course, email })` | Pick fields, never `create(req.body)`. We tested it: a client could send `createdAt: "2000-01-01"` and it would be saved |
| No try / catch                               | Express 5 sends async errors to the error handler (Session 15) |
| PUT uses `set()` + `save()`                  | Validates everything and keeps `createdAt`. `findOneAndReplace()` would reset `createdAt` |
| PATCH uses `runValidators: true`             | Without it, `{ "age": 5 }` would be saved                |
| `course: course \|\| "Not specified"` in PUT | `default` only applies when a document is created        |
| `sort` allow list                            | Only these exact values are accepted, so `req.query.sort` cannot be something strange |
| `CastError` in the error handler             | Covers bad ids and bad values like `{ "age": "abc" }` in PATCH |

---

## Testing the API

Start the server

```bash
node seed.js
node --watch server.js
```

Output

```text
Connected to MongoDB
Server running on port 3000
```

Use the same test-api.js as [Session 18](18-mongodb-crud-operations.md#testing-the-api). It works without any change, because the API looks the same from the outside

```bash
node test-api.js
```

Output (your ids and dates will be different)

```text
POST /students 201 {"success":true,"data":{"name":"Sarah","age":23,"course":"Biology","email":"sarah@example.com","_id":"6ac20a8f8da19190f34f39f9","createdAt":"2026-10-04T08:13:03.533Z","updatedAt":"2026-10-04T08:13:03.533Z","__v":0}}
POST /students 400 {"success":false,"message":"Validation failed","errors":["Age must be a number","Name is required","Email is required"]}
POST /students 409 {"success":false,"message":"Email already exists"}
GET /students/6ac20a8f8da19190f34f39f9 200 {"success":true,"data":{"_id":"6ac20a8f8da19190f34f39f9","name":"Sarah","age":23,"course":"Biology","email":"sarah@example.com","createdAt":"2026-10-04T08:13:03.533Z","updatedAt":"2026-10-04T08:13:03.533Z","__v":0}}
PATCH /students/6ac20a8f8da19190f34f39f9 200 {"success":true,"data":{"_id":"6ac20a8f8da19190f34f39f9","name":"Sarah","age":24,"course":"Biology","email":"sarah@example.com","createdAt":"2026-10-04T08:13:03.533Z","updatedAt":"2026-10-04T08:13:03.588Z","__v":0}}
PUT /students/6ac20a8f8da19190f34f39f9 400 {"success":false,"message":"Validation failed","errors":["Age is required","Email is required"]}
GET /students/123 400 {"success":false,"message":"Invalid _id: 123"}
DELETE /students/6ac20a8f8da19190f34f39f9 200 {"success":true,"message":"Student deleted successfully"}
GET /students/6ac20a8f8da19190f34f39f9 404 {"success":false,"message":"Student not found"}
```

Differences from Session 18

* Students now have `updatedAt` and `__v`. After PATCH, `updatedAt` changed by itself.
* The empty name in the 2nd request gives "Name is required" (because of `trim` and `required`), and `"abc"` gives our `cast` message.
* `GET /students/123` gives "Invalid _id: 123" from the CastError check.

More things to try

```text
GET /students?course=Mathematics&sort=-age     2 students, Jane Smith first
PATCH { "age": "abc" }                          400 Invalid age: abc
PATCH { "age": 5, "email": "nope" }             400 Please enter a valid email, Age must be at least 18
PATCH { "email": "jane@example.com" }           409 Email already exists
POST  { ..., "age": "30" }                      201, age saved as the number 30
```

---

## Beginner Mistakes

### Mistake 1

Gluing the database name onto the connection string.

`` `${MONGODB_URI}/${DB_NAME}` `` silently uses the wrong database with Atlas strings. Use `{ dbName: process.env.DB_NAME }`.

---

### Mistake 2

Forgetting `runValidators: true` on updates.

`findByIdAndUpdate()` saves `age: 5` although the schema says `min: 18`.

---

### Mistake 3

Expecting the updated document without `returnDocument: "after"`.

`findByIdAndUpdate()` returns the old document by default.

---

### Mistake 4

Using `next` in a Mongoose 9 hook.

```javascript
schema.pre("save", async function (next) { next(); });
```

Throws `TypeError: next is not a function`. Remove `next` and use `return`.

---

### Mistake 5

Using an arrow function for a hook or method.

```javascript
schema.methods.getLabel = () => this.name; // this is not the document
```

Use `function () { ... }`.

---

### Mistake 6

Expecting hooks to run on `findByIdAndUpdate()`.

Save hooks run only on `save()` and `create()`. Use find + `save()` when a hook must run.

---

### Mistake 7

Passing `req.body` straight to `create()`.

Unknown fields are dropped, but schema fields like `createdAt` can still be set by the client. Pick the fields you allow.

---

### Mistake 8

Treating `unique: true` as validation.

It creates an index. Duplicates give error code 11000, not a ValidationError, and existing duplicates stop the index from being built.

---

### Mistake 9

Adding hooks or methods after `mongoose.model()`.

The model is built from the schema at that moment. Add hooks and methods first.

---

## Practice Exercises

### Exercise 1

Create a Product model with

* name (required, at least 3 characters, trimmed)
* price (required, at least 0)
* category (required, one of "books", "electronics", "clothes")
* stock (default 0)
* timestamps

### Exercise 2

Make product names unique. Creating a product with an existing name should return 409

### Exercise 3

Build the full products API (GET, GET by id, POST, PUT, PATCH, DELETE) with the same error handler as the Student API

### Exercise 4

Add GET /products?category=books&sort=-price

### Exercise 5

Add a route to find products with low stock

GET /products/low-stock?lessThan=10

Hint: use `$lt` and `Number()`. Put this route **before** `/products/:id` (Session 13), otherwise "low-stock" is treated as an id and gives a CastError

### Exercise 6

Add a `slug` field and a pre save hook that builds it from the name, like the example in [Middleware Hooks](#middleware-hooks)

### Exercise 7

Add an instance method `isInStock()` that returns true when stock is more than 0

### Exercise 8

Create a Review model with `text`, `rating` (1 to 5) and `product` (a ref to Product). Add GET /reviews that uses `populate("product", "name price")`

### Exercise 9

Send `{ "price": -5 }` with PATCH, first without `runValidators`, then with it. Compare what is saved

---

## Interview Questions

### What is Mongoose

Mongoose is an ODM (Object Data Modeling) library for MongoDB and Node.js. It adds schemas, validation, hooks and relationships on top of the MongoDB driver

### What is the difference between a Schema and a Model

Schema defines the structure and rules of documents
Model is built from a schema and gives methods to work with a collection (find, create, ...)

### What is the difference between a Model and a Document

A model represents the whole collection (Student). A document is one record (one student) with methods like save()

### How does Mongoose name the collection

It makes the model name lowercase and plural: Student becomes students, Person becomes people

### Why do we use schemas in Mongoose

To define the structure, validation, and default values for documents

### What is the purpose of the timestamps option

It automatically adds createdAt and updatedAt fields and keeps updatedAt current

### What is __v

The version key Mongoose adds to every document. It helps detect conflicting updates

### How do you get the updated document from findByIdAndUpdate

Pass { returnDocument: "after" }. The older { new: true } is deprecated in Mongoose 9

### Does findByIdAndUpdate run validation

No, not by default. Pass { runValidators: true }

### What is the difference between create and save

create creates and saves in one step
save is called on a document you created with new Model(), or one you found and changed

### How does Mongoose handle validation

It checks the data against the schema before saving. If anything fails, it throws a ValidationError with one entry per field in err.errors

### What is a CastError

An error when a value cannot be converted to the schema type, for example findById("123") or age "abc"

### How do you handle a duplicate unique field

The unique index makes MongoDB throw error code 11000. Check err.code === 11000 and answer 409

### What is a pre save hook

A function that runs before a document is saved. this is the document. It is used for things like building a slug or hashing a password

### Why must hooks and methods use normal functions

Arrow functions do not have their own this, so they cannot access the document

### What does isModified do

It returns true if a field is new or changed, so a hook can skip work when the field did not change

### What does select: false do

It hides a field from query results unless you ask for it with .select("+field")

### What is populate

It replaces a stored ObjectId (a field with ref) with the actual document from the other collection

### What does lean() do

It returns plain JavaScript objects instead of Mongoose documents. Faster, but without methods like save()

---
