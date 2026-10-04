## Table of Contents

* [What is MongoDB](#what-is-mongodb)
* [SQL vs NoSQL](#sql-vs-nosql)
* [Important MongoDB Terms](#important-mongodb-terms)
* [How MongoDB Stores Data](#how-mongodb-stores-data)
* [Data Types in a Document](#data-types-in-a-document)
* [What is ObjectId](#what-is-objectid)
* [Installing MongoDB](#installing-mongodb)
* [MongoDB Atlas Cloud Setup](#mongodb-atlas-cloud-setup)
* [Understanding the Connection String](#understanding-the-connection-string)
* [Connecting to MongoDB](#connecting-to-mongodb)
* [Connection Errors](#connection-errors)
* [MongoDB Compass](#mongodb-compass)
* [The MongoDB Shell (mongosh)](#the-mongodb-shell-mongosh)
* [Basic MongoDB Commands](#basic-mongodb-commands)
* [Creating a Database](#creating-a-database)
* [Creating a Collection](#creating-a-collection)
* [Inserting Documents](#inserting-documents)
* [Finding Documents](#finding-documents)
* [Updating Documents](#updating-documents)
* [Deleting Documents](#deleting-documents)
* [Complete Example](#complete-example)
* [Beginner Mistakes](#beginner-mistakes)
* [Practice Exercises](#practice-exercises)
* [Interview Questions](#interview-questions)

---

## What is MongoDB

MongoDB is a database

A database stores your application data permanently

Until now we stored data in an array (Session 15)

```javascript
let students = []; // Data lost when server restarts
```

![Restart the server: the array is empty, the database still has the data](images/17-mongodb-introduction/array-vs-database.gif)

| Storing in an array          | Storing in MongoDB                     |
| ---------------------------- | -------------------------------------- |
| Data lost when server restarts | Data stays after a restart           |
| Only one server can see it   | Many servers can use the same data     |
| You write every search yourself | Built-in search and filtering       |
| Slow with a lot of data      | Fast even with millions of records     |

In Session 05 you saved data to a JSON file. That survives a restart, but two requests writing the file at the same time can lose data, and you must read the whole file to find one student. A database solves both.

MongoDB is a NoSQL database

It stores data in a JSON-like format called BSON (Binary JSON)

---

## SQL vs NoSQL

Traditional databases use SQL (Structured Query Language)

| SQL databases | NoSQL databases |
| ------------- | --------------- |
| MySQL         | MongoDB         |
| PostgreSQL    | Firebase        |
| SQL Server    | Cassandra       |
| Oracle        | Redis           |

NoSQL means "Not Only SQL". These databases do not store data in tables.

Main differences

| Feature       | SQL Database                  | MongoDB                                  |
| ------------- | ----------------------------- | ---------------------------------------- |
| Data format   | Tables with rows and columns  | Documents (like JavaScript objects)      |
| Schema        | Fixed: every row has the same columns | Flexible: documents can have different fields |
| Related data  | Separate tables joined by keys | Often stored inside the document, or linked by _id |
| Query language | SQL (`SELECT * FROM students`) | JavaScript-like objects (`find({ age: 20 })`) |

Think of it like this

![A spreadsheet row becomes a document card](images/17-mongodb-introduction/sql-vs-mongo.gif)

* SQL is like an Excel spreadsheet: rows and columns, strict format
* MongoDB is like a folder of JSON files: each file can be different

Neither is "better". MongoDB is popular with Node.js because documents look exactly like the JavaScript objects you already use.

---

## Important MongoDB Terms

You need to learn these terms

| SQL Term    | MongoDB Term | Description                    |
| ----------- | ------------ | ------------------------------ |
| Database    | Database     | Container for collections      |
| Table       | Collection   | Group of documents             |
| Row         | Document     | Single record                  |
| Column      | Field        | Key-value pair                 |
| Primary Key | _id          | Unique identifier              |

![Cluster, database, collection and document fit inside each other](images/17-mongodb-introduction/hierarchy.gif)

A **cluster** is the MongoDB server (or group of servers) itself. One cluster holds many databases.

Example of a student in SQL

| id | name | age | course |
| -- | ---- | --- | ------ |
| 1  | John | 20  | CS     |

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

```javascript
{
  _id: ObjectId("507f1f77bcf86cd799439011"),
  name: "John Doe",
  age: 20,
  course: "Computer Science",
  email: "john@example.com",
  address: {
    city: "New York",
    zip: "10001"
  },
  hobbies: ["reading", "coding", "gaming"]
}
```

Features of MongoDB documents

* Can have nested objects (address)
* Can have arrays (hobbies)
* Each document can have different fields
* _id is automatically created

Multiple documents in a collection

```javascript
// Collection: students

// Document 1
{ _id: 1, name: "John", age: 20 }

// Document 2
{ _id: 2, name: "Jane", age: 22, email: "jane@example.com" }

// Document 3
{ _id: 3, name: "Mike", course: "Physics" }
```

Notice each document can have different fields

MongoDB does not enforce a fixed structure

That freedom is also a danger: a typo like `nmae: "John"` is saved without any error. In Session 19, Mongoose adds rules (a schema) to stop this.

---

## Data Types in a Document

BSON can store more types than JSON

![Each field in a document has a type](images/17-mongodb-introduction/document-types.gif)

| Type      | Example                              |
| --------- | ------------------------------------ |
| String    | `name: "John"`                       |
| Number    | `age: 20`, `price: 9.99`             |
| Boolean   | `isActive: true`                     |
| Date      | `createdAt: new Date()`              |
| Array     | `hobbies: ["reading", "coding"]`     |
| Object    | `address: { city: "New York" }`      |
| null      | `phone: null`                        |
| ObjectId  | `_id: ObjectId("507f1f77bc...")`     |

JSON has no Date type and no ObjectId type. BSON does, so a date stays a real date in the database.

Types matter when you search. The number `5` and the string `"5"` are different values

```javascript
await collection.insertOne({ age: 5 });

await collection.findOne({ age: "5" }); // null, not found
await collection.findOne({ age: 5 });   // found
```

This is important because `req.params` and `req.query` are always strings (Session 13), and so is `process.env` (Session 16). Convert with `Number()` before searching a number field.

---

## What is ObjectId

When you insert a document without an `_id`, MongoDB creates one for you. It is an **ObjectId**.

```text
6ac203b2b7392100f72ccf05
```

An ObjectId is 24 characters long, using only 0-9 and a-f (hexadecimal)

![An ObjectId is made of a time, a random part and a counter](images/17-mongodb-introduction/objectid.gif)

| Part                    | Characters | Meaning                                  |
| ----------------------- | ---------- | ---------------------------------------- |
| Timestamp               | first 8    | When the document was created (seconds)  |
| Random value            | next 10    | Different for each machine and process   |
| Counter                 | last 6     | Goes up by one for every new ObjectId    |

Because of this, two ObjectIds are never the same, even when many servers create them at the same moment. That is why MongoDB does not use `length + 1` like our array in Session 15.

You can read the creation time back

```javascript
const { ObjectId } = require("mongodb");

const id = new ObjectId("6ac203b2b7392100f72ccf05");

console.log(id.getTimestamp());
console.log(id.toString());
```

Output

```text
2026-10-04T07:43:46.000Z
6ac203b2b7392100f72ccf05
```

An ObjectId is not a string. Searching with the string finds nothing

```javascript
await collection.findOne({ _id: "6ac203b2b7392100f72ccf05" });                // null
await collection.findOne({ _id: new ObjectId("6ac203b2b7392100f72ccf05") }); // found
```

An id from a URL like `/students/6ac203b2b7392100f72ccf05` is a string, so you must wrap it in `new ObjectId()`. A wrong id crashes

```javascript
new ObjectId("123");
```

```text
BSONError: input must be a 24 character hex string, 12 byte Uint8Array, or an integer
```

Session 18 shows how to check an id with `ObjectId.isValid()` before using it.

---

## Installing MongoDB

There are two ways to use MongoDB

| Option                        | Good for                         | Connection string                          |
| ----------------------------- | -------------------------------- | ------------------------------------------ |
| MongoDB Atlas (cloud)         | Beginners, works anywhere        | `mongodb+srv://...mongodb.net/`            |
| MongoDB Community Server (local) | Working offline               | `mongodb://127.0.0.1:27017`                |

![Atlas runs MongoDB in the cloud, Community Server runs it on your laptop](images/17-mongodb-introduction/atlas-vs-local.gif)

For beginners, MongoDB Atlas is easier

* No installation needed
* Free tier available
* Works on any computer

Let us use MongoDB Atlas

If you install MongoDB locally, it runs on port `27017` (like your Express app runs on port 3000). Use `127.0.0.1` instead of `localhost` in the connection string. On some computers `localhost` points to the IPv6 address `::1`, which MongoDB does not listen on by default.

---

## MongoDB Atlas Cloud Setup

The Atlas website changes its buttons from time to time, but the steps stay the same

![Create a free cluster, a user, allow your IP, then copy the connection string](images/17-mongodb-introduction/atlas-setup.gif)

Step 1 - Go to mongodb.com/atlas and sign up for a free account

Step 2 - Create a cluster

Choose the FREE tier (M0)

Step 3 - Choose your cloud provider and region

Any provider is fine (AWS, Google, Azure)

Choose the region closest to you

Step 4 - Create a database user

```text
Username: admin
Password: yourpassword (save this)
```

This is not your Atlas login. It is a user that your Node.js app uses to log in to the database. You can manage users later under **Database Access**.

Step 5 - Add your IP address (**Network Access**)

Click "Add My Current IP Address"

This allows your computer to connect. Atlas blocks every other computer.

Your home IP can change. If connecting suddenly stops working, check Network Access first. `0.0.0.0/0` allows every computer in the world. It is handy for testing, but your password is then the only protection.

Step 6 - Get your connection string

Click Connect, then choose Drivers (Node.js)

Copy the connection string

It looks like this

```text
mongodb+srv://admin:<db_password>@cluster0.abc123.mongodb.net/?retryWrites=true&w=majority&appName=Cluster0
```

Replace `<db_password>` with your actual password (without the `< >`)

Save this connection string in your .env file (Session 16)

---

## Understanding the Connection String

![Each part of the connection string has a job](images/17-mongodb-introduction/connection-string.gif)

```text
mongodb+srv://admin:secret123@cluster0.abc123.mongodb.net/school?retryWrites=true
```

| Part                              | Meaning                                          |
| --------------------------------- | ------------------------------------------------ |
| `mongodb+srv://`                  | Protocol. Atlas uses `+srv`, local uses `mongodb://` |
| `admin`                           | Database username                                |
| `secret123`                       | Database password                                |
| `cluster0.abc123.mongodb.net`     | Address of your cluster                          |
| `/school`                         | Default database (optional)                      |
| `?retryWrites=true`               | Extra options (like a URL query string, Session 09) |

The connection string contains your password, so it is a secret. Keep it in .env, never in your code.

Special characters in the password

`@`, `:`, `/`, `#` and `?` have a meaning in the connection string. If your password is `p@ss#1`, MongoDB cannot tell where the password ends

```text
MongoParseError: Invalid connection string
```

Fix it by encoding the password with JavaScript's built-in `encodeURIComponent()`, which turns special characters into safe codes

```javascript
console.log(encodeURIComponent("p@ss#1"));
```

Output

```text
p%40ss%231
```

So the connection string becomes `mongodb+srv://admin:p%40ss%231@...`. The easiest fix for beginners: create a database user with a password made of only letters and numbers.

---

## Connecting to MongoDB

We need a MongoDB driver to connect from Node.js. A driver is a package that knows how to talk to the database server.

![Node.js talks to the database through the driver](images/17-mongodb-introduction/driver-flow.gif)

Install the MongoDB package and dotenv

```bash
npm install mongodb dotenv
```

.env

```text
MONGODB_URI=mongodb+srv://admin:yourpassword@cluster0.abc123.mongodb.net/
DB_NAME=school
```

For a local MongoDB, use `MONGODB_URI=mongodb://127.0.0.1:27017`

connect.js

```javascript
require("dotenv").config({ quiet: true });
const { MongoClient } = require("mongodb");

// Create a new client
const client = new MongoClient(process.env.MONGODB_URI);

async function connect() {
  try {
    await client.connect();
    console.log("Connected to MongoDB");

    // Select database
    const db = client.db(process.env.DB_NAME);

    // Select collection
    const collection = db.collection("students");

    // Do operations here

  } catch (error) {
    console.error("Connection failed:", error.message);
  } finally {
    await client.close();
  }
}

connect();
```

Output

```text
Connected to MongoDB
```

New things in this code

| Code                       | Meaning                                              |
| -------------------------- | ---------------------------------------------------- |
| `const { MongoClient } = ...` | Take only MongoClient from the package (destructuring) |
| `new MongoClient(uri)`     | Create a client. It does not connect yet             |
| `await client.connect()`   | Actually connect. It takes time, so we wait (Session 03) |
| `finally { ... }`          | Runs after try or catch, whether there was an error or not |
| `client.close()`           | Close the connection so the script can end           |

`finally` is the right place for cleanup, like closing the connection. Without `client.close()`, the script keeps running because the connection is still open.

In a script, connect, work, and close. In an Express server (Session 18), you connect **once** when the server starts and keep the connection open. Never connect inside every route.

---

## Connection Errors

When connecting fails, read the error name and message. These are the real errors you will meet

![Each connection problem gives a different error](images/17-mongodb-introduction/connection-errors.gif)

| Error                                         | Cause                                     | Fix                                            |
| --------------------------------------------- | ----------------------------------------- | ---------------------------------------------- |
| `MongoServerError: bad auth : Authentication failed.` (Atlas) or `Authentication failed.` (local) | Wrong username or password | Check the database user in Database Access |
| `MongoServerSelectionError` after about 30 seconds | Atlas blocks your IP, or no internet  | Add your IP in Network Access                  |
| `MongoServerSelectionError: connect ECONNREFUSED 127.0.0.1:27017` | Local MongoDB is not running | Start the MongoDB service                 |
| `MongoParseError: Invalid scheme, expected connection string to start with "mongodb://" or "mongodb+srv://"` | Typo in the connection string, or `MONGODB_URI` is undefined | Check .env and that dotenv is loaded first |
| `MongoParseError: Invalid connection string` | Special characters in the password        | Use `encodeURIComponent()` or a simpler password |

The driver waits 30 seconds before it gives up on a server. While learning, you can make it fail faster

```javascript
const client = new MongoClient(process.env.MONGODB_URI, {
  serverSelectionTimeoutMS: 5000
});
```

---

## MongoDB Compass

MongoDB Compass is a graphical user interface for MongoDB

It lets you see your data without writing code

Download from mongodb.com/products/compass

It is free

Paste the same connection string from your .env file to connect

Once connected, you can

* See all databases
* See all collections
* View documents
* Add, edit, delete documents
* Run queries

This is very helpful for beginners

You can see what your code is doing. Run your script, then click refresh in Compass to see the new documents.

---

## The MongoDB Shell (mongosh)

mongosh is a terminal for MongoDB. You type commands and see results immediately. Compass has it built in: click the `>_ MONGOSH` bar at the bottom.

```text
use school
db.students.insertOne({ name: "John", age: 20 })
db.students.insertOne({ name: "Jane", age: 22, email: "jane@example.com" })
db.students.find()
db.students.find({ age: { $gt: 21 } })
db.students.countDocuments()
show dbs
show collections
```

Output

```text
test> use school
switched to db school
school> db.students.insertOne({ name: "John", age: 20 })
{
  acknowledged: true,
  insertedId: ObjectId('6ac203db1c7c6c24b3ccf05b')
}
school> db.students.find()
[
  { _id: ObjectId('6ac203db1c7c6c24b3ccf05b'), name: 'John', age: 20 },
  {
    _id: ObjectId('6ac203db1c7c6c24b3ccf05c'),
    name: 'Jane',
    age: 22,
    email: 'jane@example.com'
  }
]
school> db.students.find({ age: { $gt: 21 } })
[
  {
    _id: ObjectId('6ac203db1c7c6c24b3ccf05c'),
    name: 'Jane',
    age: 22,
    email: 'jane@example.com'
  }
]
school> db.students.countDocuments()
2
school> show dbs
admin    8.00 KiB
config  12.00 KiB
local    8.00 KiB
school   8.00 KiB
school> show collections
students
```

The prompt (`test>`, `school>`) shows the current database. Your ObjectIds will be different.

| Shell command      | Meaning                                  |
| ------------------ | ---------------------------------------- |
| `show dbs`         | List databases                           |
| `use school`       | Switch to the school database            |
| `show collections` | List collections in the current database |
| `db.students.find()` | Same `find` as in Node.js, no `await` or `toArray()` needed |

`admin`, `config` and `local` are MongoDB's own databases. Do not change them. (On Atlas you may not see them.)

The shell uses the same methods as the Node.js driver, so you can test a query in the shell first, then copy it into your code.

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

| Operation | Method                                 |
| --------- | -------------------------------------- |
| Create    | `insertOne`, `insertMany`              |
| Read      | `find`, `findOne`, `countDocuments`    |
| Update    | `updateOne`, `updateMany`              |
| Delete    | `deleteOne`, `deleteMany`              |

All of them return a Promise, so use `await` inside an `async` function

---

## Creating a Database

In MongoDB, you do not explicitly create a database

You just start using it

```javascript
// Points to a database named "school"
const db = client.db("school");
```

No create command needed

MongoDB creates it when you first insert data

![The database appears only after the first insert](images/17-mongodb-introduction/lazy-create.gif)

We tested this: right after `client.db("school")` and `db.collection("students")`, the list of databases is still `admin, config, local`. After the first `insertOne`, `school` appears.

This also means a typo creates a new database silently. `client.db("shcool")` gives you an empty database, not an error. If your data seems missing, check the name in Compass.

---

## Creating a Collection

Same as database

You do not explicitly create a collection

```javascript
// Points to a collection named "students"
const collection = db.collection("students");
```

MongoDB creates the collection when you first insert a document

Collection names are usually plural and lowercase: `students`, `products`, `orders`

---

## Inserting Documents

![insertOne adds a document card and returns its new _id](images/17-mongodb-introduction/crud.gif)

Insert one document

```javascript
const result = await collection.insertOne({
  name: "John Doe",
  age: 20,
  course: "Computer Science",
  email: "john@example.com"
});

console.log(result);
```

Output

```text
{
  acknowledged: true,
  insertedId: new ObjectId('6ac203b2b7392100f72ccf05')
}
```

`acknowledged: true` means the server confirmed the write. `insertedId` is the new `_id`.

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

Output

```text
Inserted count: 2
```

Two documents with the same `_id` are not allowed

```text
MongoServerError: E11000 duplicate key error collection: school.students index: _id_ dup key: { _id: 1 }
```

`E11000` always means "this value must be unique and already exists". You will see it again with unique emails in Session 19.

---

## Finding Documents

Find all documents

```javascript
const allStudents = await collection.find({}).toArray();
console.log(allStudents);
```

`find()` does not return the documents directly. It returns a **cursor**, a pointer that fetches documents in batches. `toArray()` collects them all into a normal array.

The `{}` is a filter. An empty filter matches every document.

Find documents with filter

```javascript
// Find students with age 20
const result = await collection.find({ age: 20 }).toArray();

// Find students in Computer Science course
const csStudents = await collection.find({ course: "Computer Science" }).toArray();

// Find one student by name
const john = await collection.findOne({ name: "John Doe" });
```

| Method      | Returns                     | If nothing matches |
| ----------- | --------------------------- | ------------------ |
| `find()`    | Cursor (use `toArray()`)    | Empty array `[]`   |
| `findOne()` | One document                | `null`             |

`findOne` does not need `toArray()`

Find with conditions

```javascript
// Age greater than 20
const older = await collection.find({ age: { $gt: 20 } }).toArray();

// Age less than or equal to 21
const younger = await collection.find({ age: { $lte: 21 } }).toArray();

// Course is Computer Science AND age is 20
const both = await collection.find({
  course: "Computer Science",
  age: 20
}).toArray();

// How many students
const count = await collection.countDocuments();
```

Words starting with `$` are **operators**. `$gt` means "greater than", `$lte` means "less than or equal". Session 18 covers all of them.

---

## Updating Documents

Update one document

```javascript
const result = await collection.updateOne(
  { name: "John Doe" },  // Find document
  { $set: { age: 21 } }  // Update age
);

console.log(result);
```

Output

```text
{
  acknowledged: true,
  modifiedCount: 1,
  upsertedId: null,
  upsertedCount: 0,
  matchedCount: 1
}
```

| Field           | Meaning                                         |
| --------------- | ----------------------------------------------- |
| `matchedCount`  | How many documents matched the filter           |
| `modifiedCount` | How many were actually changed                  |

![matchedCount and modifiedCount tell different stories](images/17-mongodb-introduction/update-counts.gif)

| Situation                               | matchedCount | modifiedCount |
| --------------------------------------- | ------------ | ------------- |
| John found, age changed 20 to 21        | 1            | 1             |
| Run it again (age is already 21)        | 1            | 0             |
| Nobody named that                       | 0            | 0             |

To answer "was it found?" (404 or not), check `matchedCount`, not `modifiedCount`.

`$set` is required. It changes only the listed fields and keeps the others. Without it, the driver refuses

```javascript
await collection.updateOne({ name: "John Doe" }, { age: 30 });
```

```text
MongoInvalidArgumentError: Update document requires atomic operators
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

If no Sarah exists, a new document `{ name: "Sarah", age: 23, course: "Biology" }` is created, and `result.upsertedId` holds its new `_id`.

---

## Deleting Documents

Delete one document

```javascript
const result = await collection.deleteOne({ name: "John Doe" });
console.log("Deleted count:", result.deletedCount);
```

Output

```text
Deleted count: 1
```

`deletedCount` is `0` if nothing matched

Delete multiple documents

```javascript
const result = await collection.deleteMany({ age: { $lt: 18 } });
console.log("Deleted count:", result.deletedCount);
```

Delete all documents in a collection

```javascript
const result = await collection.deleteMany({});
```

Be careful: an empty filter `{}` matches every document. There is no undo.

Drop entire collection

```javascript
await collection.drop();
```

`deleteMany({})` empties the collection. `drop()` removes the collection itself.

---

## Complete Example

Project structure

```text
mongo-demo/
├── .env
├── .gitignore
├── package.json
└── app.js
```

.env

```text
MONGODB_URI=mongodb+srv://admin:yourpassword@cluster0.abc123.mongodb.net/
DB_NAME=school
```

app.js

```javascript
require("dotenv").config({ quiet: true });
const { MongoClient } = require("mongodb");

async function main() {
  const client = new MongoClient(process.env.MONGODB_URI);

  try {
    // Connect to MongoDB
    await client.connect();
    console.log("Connected to MongoDB");

    // Get database and collection
    const db = client.db(process.env.DB_NAME);
    const students = db.collection("students");

    // INSERT - Add a new student
    const insertResult = await students.insertOne({
      name: "John Doe",
      age: 20,
      course: "Computer Science",
      email: "john@example.com",
      createdAt: new Date()
    });
    console.log("Inserted student with id:", insertResult.insertedId.toString());

    // FIND - Get all students
    const allStudents = await students.find({}).toArray();
    console.log("Number of students:", allStudents.length);

    // FIND - Get one student
    const john = await students.findOne({ _id: insertResult.insertedId });
    console.log("Found:", john.name, "age", john.age);

    // UPDATE - Change John's age
    const updateResult = await students.updateOne(
      { _id: insertResult.insertedId },
      { $set: { age: 21 } }
    );
    console.log("Matched:", updateResult.matchedCount, "Modified:", updateResult.modifiedCount);

    // FIND again to see the change
    const updatedJohn = await students.findOne({ _id: insertResult.insertedId });
    console.log("Updated age:", updatedJohn.age);

    // DELETE - Remove John
    const deleteResult = await students.deleteOne({ _id: insertResult.insertedId });
    console.log("Deleted:", deleteResult.deletedCount);

  } catch (error) {
    console.error("Error:", error.message);
  } finally {
    // Close connection
    await client.close();
    console.log("Connection closed");
  }
}

main();
```

Run it

```bash
node app.js
```

Output (your id will be different)

```text
Connected to MongoDB
Inserted student with id: 6ac2046f9b0dc0cddb775083
Number of students: 1
Found: John Doe age 20
Matched: 1 Modified: 1
Updated age: 21
Deleted: 1
Connection closed
```

`insertedId` is already an ObjectId, so we can use it directly in `{ _id: insertResult.insertedId }`. Searching by `_id` is the safest way to find one exact document, because two students can have the same name.

To see the document in Compass, comment out the DELETE step, run the script again, and refresh Compass.

---

## Beginner Mistakes

### Mistake 1

Searching by `_id` with a string.

```javascript
collection.findOne({ _id: req.params.id }); // always null
```

Wrap it: `{ _id: new ObjectId(req.params.id) }`.

---

### Mistake 2

Forgetting `toArray()` after `find()`.

```javascript
const students = await collection.find({});
console.log(students.length); // undefined
```

`find()` gives a cursor, not an array. Use `await collection.find({}).toArray()`.

---

### Mistake 3

Forgetting `await`.

```javascript
const john = collection.findOne({ name: "John" });
console.log(john.name); // undefined, john is a Promise
```

---

### Mistake 4

Updating without `$set`.

The driver throws `Update document requires atomic operators`. Write `{ $set: { age: 21 } }`.

---

### Mistake 5

Searching a number field with a string.

`{ age: "20" }` does not match `age: 20`. Use `Number(req.query.age)`.

---

### Mistake 6

Leaving `<db_password>` in the connection string, or using a password with `@` or `#` without encoding it.

---

### Mistake 7

Typo in the database or collection name.

`client.db("shcool")` creates a new empty database without any error. Check the names in Compass.

---

### Mistake 8

Putting the connection string in the code and pushing it to GitHub.

It contains your password. Keep it in .env (Session 16).

---

## Practice Exercises

### Exercise 1

Create a database named "library"

Create a collection named "books"

Insert 3 books with

* title
* author
* year
* price

Check them in Compass

### Exercise 2

Find all books published after 2010

### Exercise 3

Update a book's price. Print `matchedCount` and `modifiedCount`. Run it twice. What changes?

### Exercise 4

Delete a book by its title

### Exercise 5

Find all books by a specific author

### Exercise 6

Insert a book, print its `insertedId`, then find it again using `new ObjectId("...")` with the id as a string. Then try the plain string. What happens?

### Exercise 7

Print the creation time of a book using `_id.getTimestamp()`

### Exercise 8

Run the same queries in mongosh (inside Compass) and compare the output with your Node.js script

---

## Interview Questions

### What is MongoDB

MongoDB is a NoSQL database that stores data in JSON-like documents

### What is the difference between SQL and MongoDB

SQL uses tables and rows with a fixed schema
MongoDB uses documents and collections with a flexible schema

### What is a collection in MongoDB

A collection is a group of documents, similar to a table in SQL

### What is a document in MongoDB

A document is a single record, similar to a row in SQL

### What is BSON

Binary JSON. The format MongoDB uses to store documents. It supports extra types like Date and ObjectId

### What is _id in MongoDB

_id is a unique identifier for each document. If you do not set it, MongoDB creates an ObjectId

### What is inside an ObjectId

12 bytes (24 hex characters): a timestamp, a random value and a counter. So it is unique and you can get the creation time from it

### Why does findOne({ _id: "..." }) return null

Because _id is an ObjectId, not a string. Use new ObjectId(id)

### What is MongoDB Atlas

MongoDB Atlas is the cloud version of MongoDB hosted by MongoDB

### Do you need to create a database before using it

No
MongoDB creates the database and collection when you first insert data

### What is the difference between find and findOne

find returns a cursor (use toArray to get an array, empty if nothing matches). findOne returns one document or null

### What is the difference between matchedCount and modifiedCount

matchedCount is how many documents matched the filter. modifiedCount is how many were actually changed. A document that already had the new value is matched but not modified

### What is upsert

An update that inserts a new document if no document matches the filter

### What is the difference between deleteMany({}) and drop()

deleteMany({}) removes all documents but keeps the collection. drop() removes the collection itself

### What does E11000 mean

Duplicate key error. A value that must be unique (like _id) already exists

---
