## Table of Contents

* [What is Authentication](#what-is-authentication)
* [What is Authorization](#what-is-authorization)
* [Sessions vs Tokens](#sessions-vs-tokens)
* [What is JWT](#what-is-jwt)
* [How JWT Works](#how-jwt-works)
* [JWT Structure](#jwt-structure)
* [Why the Signature Stops Cheating](#why-the-signature-stops-cheating)
* [Setting Up the Project](#setting-up-the-project)
* [Creating the User Model](#creating-the-user-model)
* [Creating JWT Token](#creating-jwt-token)
* [Register User](#register-user)
* [Login User](#login-user)
* [Verifying JWT Token](#verifying-jwt-token)
* [Role-Based Authorization](#role-based-authorization)
* [Making the First Admin](#making-the-first-admin)
* [Protected Routes](#protected-routes)
* [Complete Authentication API](#complete-authentication-api)
* [Testing the API](#testing-the-api)
* [Where the Client Keeps the Token](#where-the-client-keeps-the-token)
* [Logging Out](#logging-out)
* [Beginner Mistakes](#beginner-mistakes)
* [Practice Exercises](#practice-exercises)
* [Interview Questions](#interview-questions)

---

## What is Authentication

Authentication is verifying who someone is

It answers the question "Are you who you say you are?"

Think of it like showing your passport at airport security

1. You show your passport
2. Security checks it
3. Security confirms it is you
4. You are authenticated

In web applications, authentication is usually done with

* Email and password
* Username and password
* "Sign in with Google" or another account

Example

* The user enters email and password
* The server checks if they match a saved user
* If they match, the user is authenticated
* If not, authentication fails with 401

---

## What is Authorization

Authorization is what someone is allowed to do

It answers the question "What can you do?"

Think of it like a hotel room key

![The key card opens your room, but not other rooms or the staff area](images/22-authentication-using-jwt/hotel-key.gif)

* The guest gets a key card at reception (authentication)
* The key only opens their own room (authorization)
* It cannot open other rooms or staff areas

In web applications, authorization controls

* Which pages a user can see
* Which data a user can access
* Which actions a user can perform

Difference between Authentication and Authorization

| Authentication          | Authorization            |
| ----------------------- | ------------------------ |
| Who are you?            | What can you do?         |
| Happens first           | Happens after authentication |
| Login with password     | Check the user's role    |
| Fails with 401 Unauthorized | Fails with 403 Forbidden |

401 means "I don't know who you are" (no token, bad token). 403 means "I know who you are, but you are not allowed" (Session 14).

Example

* Authentication: you logged in as John
* Authorization: John can view his own profile but cannot delete other users

---

## Sessions vs Tokens

HTTP is stateless (Session 15): every request stands alone, and the server does not remember the previous one. So after login, how does the server know who is sending the next request?

| Way          | After login, the server...                     | Each request carries   |
| ------------ | ---------------------------------------------- | ---------------------- |
| Session      | Stores "session 81f3 = John" in memory or a database | A session id (cookie) |
| Token (JWT)  | Stores nothing. It gives John a signed token   | The token itself       |

With a session, the server must look up the session on every request, and all servers must share the session store. With a JWT, any server that knows the secret can check the token on its own. That is why JWTs are popular for APIs and mobile apps.

---

## What is JWT

JWT stands for JSON Web Token

It is a small, signed piece of text that says who the user is

Think of it like a visitor badge

* You show your ID at the front desk once
* Security gives you a visitor badge with your name and an end time
* You wear the badge everywhere
* Each door checks the badge, not your ID again

JWT works the same way

* The user logs in once with email and password
* The server creates a JWT and sends it to the user
* The user sends the token with every request
* The server checks the token and allows or denies access

JWT is stateless

The server does not store any session information

The information is inside the token, protected by a signature

---

## How JWT Works

![Log in once, receive a token, then send it with every request](images/22-authentication-using-jwt/jwt-flow.gif)

| Step | Who     | What happens                                        |
| ---- | ------- | --------------------------------------------------- |
| 1    | Client  | `POST /api/auth/login` with email and password      |
| 2    | Server  | Checks the password, creates a JWT with `jwt.sign()` |
| 3    | Server  | Sends the token in the response                     |
| 4    | Client  | Saves the token                                     |
| 5    | Client  | Sends it with every request in the `Authorization` header |
| 6    | Server  | Checks it with `jwt.verify()`                       |
| 7    | Server  | Allows (200) or denies (401 / 403)                  |

Example request with token

```text
GET /api/auth/me
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpZCI6...
```

The token is sent in the `Authorization` header, after the word `Bearer` and a space. "Bearer" means "whoever holds (bears) this token".

---

## JWT Structure

A JWT token has three parts separated by dots

```text
header.payload.signature
```

A real token created in this session

```text
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpZCI6IjZhYzIxMmFhZTA5NTA0OGY2YzQ2N2UwOSIsImlhdCI6MTc5MTEwMzY1OCwiZXhwIjoxNzkxNzA4NDU4fQ.0wxQF_fwNkxICdk8RT8vG6IOBerA1GOrCVxSrRdkWsc
```

![The three parts of a token decode into header, payload and signature](images/22-authentication-using-jwt/jwt-parts.gif)

The first two parts are only **base64url encoded**, not encrypted. Anyone can read them

```javascript
const [header, payload] = token.split(".");

console.log(Buffer.from(header, "base64url").toString());
console.log(Buffer.from(payload, "base64url").toString());
```

Output

```text
{"alg":"HS256","typ":"JWT"}
{"id":"6ac212aae095048f6c467e09","iat":1791103658,"exp":1791708458}
```

| Part      | Contains                                    | Example                    |
| --------- | ------------------------------------------- | -------------------------- |
| Header    | The signing algorithm                       | `"alg": "HS256"`           |
| Payload   | The data (called **claims**)                | `"id": "6ac212aa..."`      |
| Signature | Proof that the server made this token       | `0wxQF_fwNkx...`           |

Standard claims added by the library

| Claim | Meaning                          | Example      |
| ----- | -------------------------------- | ------------ |
| `iat` | Issued at (seconds since 1970)   | `1791103658` |
| `exp` | Expires at (seconds since 1970)  | `1791708458` |

`exp - iat` is 604800 seconds = 7 days.

Because anyone can read the payload, **never put secrets in it**: no password, no credit card. Put only what is needed to find the user, like the id.

You can paste any token into the website jwt.io to see its parts.

---

## Why the Signature Stops Cheating

If anyone can read and change the payload, why can't a user change `"role": "user"` to `"role": "admin"`?

![Changing the payload breaks the signature, so verify() rejects the token](images/22-authentication-using-jwt/signature-check.gif)

The signature is calculated from the header, the payload and the **secret** (`JWT_SECRET`), which only the server knows

```text
signature = HMAC-SHA256(header + "." + payload, JWT_SECRET)
```

When a token comes back, `jwt.verify()` calculates the signature again and compares. We tested what happens

| Token                                         | `jwt.verify()` result                       |
| --------------------------------------------- | ------------------------------------------- |
| Correct token                                 | The payload                                 |
| Payload changed by the user                   | `JsonWebTokenError: invalid signature`      |
| Signed with a different secret                | `JsonWebTokenError: invalid signature`      |
| Not a token at all (`"abc"`)                  | `JsonWebTokenError: jwt malformed`          |
| Valid, but `exp` is in the past               | `TokenExpiredError: jwt expired`            |
| Header changed to `"alg": "none"`, no signature | `JsonWebTokenError: jwt signature is required` |

To fake a token, an attacker needs the secret. That is why `JWT_SECRET` must be long, random and never committed (Session 16).

---

## Setting Up the Project

Create a new project

```bash
mkdir jwt-auth-api
cd jwt-auth-api
npm init -y
```

Install packages

```bash
npm install express mongoose dotenv bcryptjs jsonwebtoken
```

| Package       | Job                                 |
| ------------- | ----------------------------------- |
| express       | Web framework                       |
| mongoose      | MongoDB models (Session 19)         |
| dotenv        | Environment variables (Session 16)  |
| bcryptjs      | Hash passwords (Session 23 explains it in detail) |
| jsonwebtoken  | Create and verify JWTs              |

Create a random secret. Run this once in the terminal

```bash
node -e "console.log(require('crypto').randomBytes(32).toString('hex'))"
```

It prints 64 random characters, like `8f3a1c9e7b2d4f6a0e5c8b1d3f7a9c2e4b6d8f0a1c3e5b7d9f2a4c6e8b0d1f3a`. `crypto` is a core module (Session 04) that makes secure random values.

Create .env file

```text
PORT=5000
MONGODB_URI=mongodb+srv://yourusername:yourpassword@cluster0.abc123.mongodb.net/
DB_NAME=auth_demo
JWT_SECRET=8f3a1c9e7b2d4f6a0e5c8b1d3f7a9c2e4b6d8f0a1c3e5b7d9f2a4c6e8b0d1f3a
JWT_EXPIRES_IN=7d
```

JWT_SECRET is very important

* Use a long random value, not a word like `secret`
* Keep it in .env, never in the code and never on GitHub
* If it leaks, every token can be faked. Change it at once (this also logs everyone out)

Project structure

```text
jwt-auth-api/
├── models/
│   ├── User.js
│   └── Student.js
├── controllers/
│   ├── authController.js
│   └── studentController.js
├── middleware/
│   ├── auth.js             protect + authorize
│   └── errorHandler.js
├── routes/
│   ├── authRoutes.js
│   └── studentRoutes.js
├── make-admin.js
├── test-api.js
├── .env
└── server.js
```

---

## Creating the User Model

Create models/User.js

```javascript
const mongoose = require("mongoose");
const bcrypt = require("bcryptjs");

const userSchema = new mongoose.Schema(
  {
    name: {
      type: String,
      required: [true, "Name is required"],
      trim: true,
      minlength: [2, "Name must be at least 2 characters"]
    },
    email: {
      type: String,
      required: [true, "Email is required"],
      unique: true,
      lowercase: true,
      trim: true,
      match: [/^\S+@\S+\.\S+$/, "Please enter a valid email"]
    },
    password: {
      type: String,
      required: [true, "Password is required"],
      minlength: [6, "Password must be at least 6 characters"],
      select: false // Never returned by queries unless asked for
    },
    role: {
      type: String,
      enum: ["user", "admin"],
      default: "user"
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

// Hash the password before saving (Mongoose 9: no next)
userSchema.pre("save", async function () {
  // Only hash when the password is new or changed
  if (!this.isModified("password")) {
    return;
  }

  this.password = await bcrypt.hash(this.password, 10);
});

// Check a typed password against the saved hash
userSchema.methods.comparePassword = function (enteredPassword) {
  return bcrypt.compare(enteredPassword, this.password);
};

module.exports = mongoose.model("User", userSchema);
```

Important parts explained

| Code                                   | Meaning                                                   |
| -------------------------------------- | --------------------------------------------------------- |
| `select: false` on password            | `User.find()` never returns the password (Session 19)     |
| `pre("save", ...)`                     | Hashes the password automatically before every save (Session 19 hooks) |
| `this.isModified("password")`          | Do not hash again when only the name changes, or the hash would be hashed |
| `bcrypt.hash(password, 10)`            | Turns `"admin123"` into a hash like `$2b$10$ZFMn...` (60 characters) |
| `comparePassword()`                    | An instance method (Session 21) that checks a typed password |

What the database really stores (we checked)

```text
{ email: 'admin@example.com', password: '$2b$10$ZFMnW8k18sTtmHvbkGutmuoZGXsk5b5d8dcXDMQClw3Kkoz5A1C8m', role: 'admin' }
```

Even if the database is stolen, the real password is not in it. Session 23 explains hashing, salt and the number 10.

**Mongoose 9 warning:** many tutorials write this hook as `async function (next) { ... next(); }`. We tested that version: every registration fails with `TypeError: next is not a function` and no user is saved. Write hooks without `next` (Session 19).

---

## Creating JWT Token

Create controllers/authController.js. First the helpers

```javascript
const jwt = require("jsonwebtoken");
const User = require("../models/User");

// Create a token that says "this is user <id>"
function generateToken(id) {
  return jwt.sign({ id }, process.env.JWT_SECRET, {
    expiresIn: process.env.JWT_EXPIRES_IN || "7d"
  });
}

// The user data we send back (never the password)
function publicUser(user) {
  return { id: user._id, name: user.name, email: user.email, role: user.role };
}
```

`jwt.sign(payload, secret, options)`

| Argument                  | Meaning                                      |
| ------------------------- | -------------------------------------------- |
| `{ id }`                  | The payload: just the user's id              |
| `process.env.JWT_SECRET`  | The secret used for the signature            |
| `{ expiresIn: "7d" }`     | Adds `exp`, 7 days from now                  |

Why only the id, not the role? If the role were in the token and an admin made someone a normal user, their old token would still say "admin" for 7 days. With only the id, the server reads the current role from the database on every request.

`expiresIn` values (tested)

| Value      | Token is valid for     |
| ---------- | ---------------------- |
| `"15m"`    | 15 minutes             |
| `"1h"`     | 1 hour                 |
| `"7d"`     | 7 days                 |
| `3600` (a number) | 3600 seconds = 1 hour |
| `"3600"` (a string) | **3.6 seconds!** |

The last row is a real trap. Values from .env are always strings (Session 16), and a string without a unit is read as milliseconds. `JWT_EXPIRES_IN=3600` gives tokens that expire almost immediately. Always write a unit: `1h`, `7d`.

If `JWT_SECRET` is missing, `jwt.sign()` throws `secretOrPrivateKey must have a value`. server.js checks for it at startup.

`publicUser()` picks the fields we send back. `User.create()` returns the new document **with** the hashed password in memory (`select: false` only affects queries), so never send the whole user.

---

## Register User

```javascript
// POST /api/auth/register
const register = async (req, res) => {
  const { name, email, password } = req.body || {};

  // role is NOT taken from the body: everyone who registers is a normal user
  const user = await User.create({ name, email, password });

  res.status(201).json({ success: true, token: generateToken(user._id), user: publicUser(user) });
};
```

![Register: validate, hash the password, save, and return a token](images/22-authentication-using-jwt/register.gif)

Response

```json
{
  "success": true,
  "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpZCI6IjZhYzIxMmFh...",
  "user": {
    "id": "6ac212aae095048f6c467e09",
    "name": "Admin",
    "email": "admin@example.com",
    "role": "user"
  }
}
```

Things to notice

* **The role is never read from the body.** Many tutorials write `role: role || "user"`. Then anyone can send `"role": "admin"` and become an admin. We tested our version: Sara sent `"role": "admin"` and still got `"user"`.
* The password is hashed by the model hook, not here.
* A duplicate email is not checked with `findOne()` first. The unique index throws E11000, and the error handler answers 409 (Session 18).
* Validation errors (short password, bad email) become 400 in the error handler.
* The new user gets a token right away, so they are logged in after registering.

---

## Login User

```javascript
// POST /api/auth/login
const login = async (req, res) => {
  const { email, password } = req.body || {};

  if (!email || !password) {
    return res.status(400).json({ success: false, message: "Please provide email and password" });
  }

  // password has select: false, so ask for it here
  const user = await User.findOne({ email: String(email).toLowerCase() }).select("+password");

  // Same message for "no such email" and "wrong password"
  if (!user || !(await user.comparePassword(password))) {
    return res.status(401).json({ success: false, message: "Invalid email or password" });
  }

  if (!user.isActive) {
    return res.status(403).json({ success: false, message: "Your account has been deactivated" });
  }

  res.status(200).json({ success: true, token: generateToken(user._id), user: publicUser(user) });
};
```

![Login: find the user, compare the password, return a token or 401](images/22-authentication-using-jwt/login.gif)

| Code                                  | Why                                                      |
| ------------------------------------- | -------------------------------------------------------- |
| `.select("+password")`                | The password is hidden by default, but we need it to compare |
| `String(email).toLowerCase()`         | Emails are saved in lowercase. `String()` stops NoSQL injection (below) |
| Same message for both failures        | An attacker cannot find out which emails are registered  |
| `!user \|\| !(await ...)`             | If there is no user, `\|\|` stops before calling comparePassword |
| 403 for deactivated                   | The password is right, but the account is not allowed    |

**NoSQL injection.** A JSON body can contain objects, not only strings. We tested sending `{ "email": { "$gt": "" }, "password": "x" }`

| Code                                       | Result                                               |
| ------------------------------------------ | ---------------------------------------------------- |
| `User.findOne({ email })`                  | `{ email: { $gt: "" } }` matches **the first user**, here the admin |
| `User.findOne({ email: String(email) })`   | Searches for the text `"[object Object]"`, finds nobody |

The wrong password still stopped the attack in our test, but a query that a client can rewrite is a serious hole. Convert user input to the type you expect.

---

## Verifying JWT Token

Middleware checks the token before protected routes run (Session 14)

Create middleware/auth.js

```javascript
const jwt = require("jsonwebtoken");
const User = require("../models/User");

// Only let requests with a valid token through
const protect = async (req, res, next) => {
  const header = req.headers.authorization || "";

  // Expected format: "Bearer <token>"
  if (!header.startsWith("Bearer ")) {
    return res.status(401).json({ success: false, message: "Not logged in. Please send a token." });
  }
  const token = header.split(" ")[1];

  let decoded;
  try {
    decoded = jwt.verify(token, process.env.JWT_SECRET, { algorithms: ["HS256"] });
  } catch (err) {
    const message = err.name === "TokenExpiredError"
      ? "Your token has expired. Please log in again."
      : "Invalid token. Please log in again.";
    return res.status(401).json({ success: false, message });
  }

  // The user may have been deleted or deactivated after the token was made
  const user = await User.findById(decoded.id);
  if (!user || !user.isActive) {
    return res.status(401).json({ success: false, message: "This user no longer exists or is deactivated" });
  }

  req.user = user;
  next();
};

// Only let some roles through (use after protect)
const authorize = (...roles) => {
  return (req, res, next) => {
    if (!roles.includes(req.user.role)) {
      return res.status(403).json({ success: false, message: `Role "${req.user.role}" is not allowed to do this` });
    }
    next();
  };
};

module.exports = { protect, authorize };
```

![protect checks the header, the signature, the expiry and the user, then sets req.user](images/22-authentication-using-jwt/protect-checks.gif)

The protect middleware, step by step

| Step | Check                                | Fails with                                    |
| ---- | ------------------------------------ | --------------------------------------------- |
| 1    | Header starts with `"Bearer "`       | 401 Not logged in. Please send a token.       |
| 2    | `jwt.verify()`: signature correct?   | 401 Invalid token. Please log in again.       |
| 3    | `jwt.verify()`: not expired?         | 401 Your token has expired. Please log in again. |
| 4    | User still exists and is active      | 401 This user no longer exists or is deactivated |
| 5    | Put the user in `req.user`, `next()` | -                                             |

Why step 4? A token stays valid until it expires, even if the user was deleted or blocked. Loading the user on every request lets you block someone immediately (we tested: a deactivated user's valid token got 401).

`{ algorithms: ["HS256"] }` tells verify to accept only the algorithm we use. It is a small extra safety line recommended by the library.

The try / catch here is needed: `jwt.verify()` throws when a token is bad, and we want a clear 401, not the generic 500 from the error handler.

`header.split(" ")[1]` splits `"Bearer eyJ..."` at the space and takes the second part, the token.

---

## Role-Based Authorization

`authorize` is a middleware factory (Session 14): a function that returns a middleware

```javascript
router.post("/", authorize("admin"), createStudent);
```

`...roles` is a **rest parameter**: it collects all arguments into an array. `authorize("admin", "teacher")` gives `roles = ["admin", "teacher"]`.

![authorize lets the admin through and stops the normal user with 403](images/22-authentication-using-jwt/authorize.gif)

| Request                     | protect             | authorize("admin")  | Result |
| --------------------------- | ------------------- | ------------------- | ------ |
| No token                    | stops               | -                   | 401    |
| Sara's token (role user)    | passes, req.user = Sara | stops           | 403    |
| Admin's token               | passes              | passes              | 200 / 201 |

`authorize` must come **after** `protect`, because it reads `req.user`, which protect sets.

---

## Making the First Admin

Nobody can register as admin, which is correct. So how do you get the first admin? A small script, run by the developer on the server, changes the role in the database

make-admin.js

```javascript
// Usage: node make-admin.js someone@example.com
require("dotenv").config({ quiet: true });
const mongoose = require("mongoose");
const User = require("./models/User");

async function makeAdmin() {
  const email = process.argv[2];

  if (!email) {
    console.log("Usage: node make-admin.js <email>");
    return;
  }

  try {
    await mongoose.connect(process.env.MONGODB_URI, { dbName: process.env.DB_NAME });

    const user = await User.findOneAndUpdate(
      { email: email.toLowerCase() },
      { role: "admin" },
      { returnDocument: "after" }
    );

    console.log(user ? `${user.email} is now an admin` : `No user with email ${email}`);
  } finally {
    await mongoose.disconnect();
  }
}

makeAdmin();
```

Register normally, then run

```bash
node make-admin.js admin@example.com
```

Output

```text
admin@example.com is now an admin
```

`process.argv[2]` is the first word after the file name (Session 07). Only someone with access to the server and the .env file can run this, which is exactly who should create admins.

---

## Protected Routes

Finish authController.js with getMe and the exports

```javascript
// GET /api/auth/me (protected)
const getMe = (req, res) => {
  // protect already loaded the user into req.user
  res.status(200).json({ success: true, user: publicUser(req.user) });
};

module.exports = { register, login, getMe };
```

routes/authRoutes.js

```javascript
const express = require("express");
const router = express.Router();

const { register, login, getMe } = require("../controllers/authController");
const { protect, authorize } = require("../middleware/auth");

// Public routes (no token needed)
router.post("/register", register);
router.post("/login", login);

// Protected route (token needed)
router.get("/me", protect, getMe);

// Admin only
router.get("/admin", protect, authorize("admin"), (req, res) => {
  res.json({ success: true, message: `Welcome ${req.user.name}, you are an admin` });
});

module.exports = router;
```

Now protect the students. models/Student.js

```javascript
const mongoose = require("mongoose");

const studentSchema = new mongoose.Schema(
  {
    name: { type: String, required: [true, "Name is required"], trim: true },
    age: { type: Number, required: [true, "Age is required"], min: 18, max: 60 },
    course: { type: String, required: [true, "Course is required"], trim: true },
    createdBy: { type: mongoose.Schema.Types.ObjectId, ref: "User" }
  },
  { timestamps: true }
);

module.exports = mongoose.model("Student", studentSchema);
```

controllers/studentController.js

```javascript
const Student = require("../models/Student");

// GET /api/students (any logged-in user)
const getAllStudents = async (req, res) => {
  const students = await Student.find().sort("name").populate("createdBy", "name");

  res.status(200).json({ success: true, count: students.length, data: students });
};

// POST /api/students (admin only)
const createStudent = async (req, res) => {
  const { name, age, course } = req.body || {};

  // Remember who created it: req.user comes from protect
  const student = await Student.create({ name, age, course, createdBy: req.user._id });

  res.status(201).json({ success: true, data: student });
};

// DELETE /api/students/:id (admin only)
const deleteStudent = async (req, res) => {
  const student = await Student.findByIdAndDelete(req.params.id);

  if (!student) {
    return res.status(404).json({ success: false, message: "Student not found" });
  }

  res.status(200).json({ success: true, message: "Student deleted successfully" });
};

module.exports = { getAllStudents, createStudent, deleteStudent };
```

`createdBy: req.user._id` records which admin created the student. The id comes from the verified token, so the client cannot fake it. `populate("createdBy", "name")` shows the admin's name in the list (Session 19).

routes/studentRoutes.js

```javascript
const express = require("express");
const router = express.Router();

const { getAllStudents, createStudent, deleteStudent } = require("../controllers/studentController");
const { protect, authorize } = require("../middleware/auth");

// Every route below needs a valid token
router.use(protect);

router.get("/", getAllStudents);
router.post("/", authorize("admin"), createStudent);
router.delete("/:id", authorize("admin"), deleteStudent);

module.exports = router;
```

`router.use(protect)` runs protect for every route in this router, so you cannot forget it on a new route.

---

## Complete Authentication API

middleware/errorHandler.js

```javascript
function notFound(req, res) {
  res.status(404).json({ success: false, message: `Route ${req.method} ${req.originalUrl} not found` });
}

function errorHandler(err, req, res, next) {
  if (err.type === "entity.parse.failed") {
    return res.status(400).json({ success: false, message: "Request body is not valid JSON" });
  }
  if (err.name === "ValidationError") {
    const errors = Object.values(err.errors).map((e) => e.message);
    return res.status(400).json({ success: false, message: "Validation failed", errors });
  }
  if (err.name === "CastError") {
    return res.status(400).json({ success: false, message: `Invalid ${err.path}: ${err.value}` });
  }
  if (err.code === 11000) {
    return res.status(409).json({ success: false, message: "An account with this email already exists" });
  }

  console.error(err);
  res.status(500).json({ success: false, message: "Something went wrong on the server" });
}

module.exports = { notFound, errorHandler };
```

server.js

```javascript
require("dotenv").config({ quiet: true });
const express = require("express");
const mongoose = require("mongoose");

const authRoutes = require("./routes/authRoutes");
const studentRoutes = require("./routes/studentRoutes");
const { notFound, errorHandler } = require("./middleware/errorHandler");

// Stop at once if the secret is missing (Session 16)
if (!process.env.JWT_SECRET) {
  console.error("FATAL ERROR: JWT_SECRET is not defined in .env");
  process.exit(1);
}

const app = express();
const PORT = process.env.PORT || 5000;

app.use(express.json());

app.get("/", (req, res) => {
  res.json({
    message: "JWT Authentication API",
    endpoints: {
      register: "POST /api/auth/register",
      login: "POST /api/auth/login",
      me: "GET /api/auth/me (token needed)",
      admin: "GET /api/auth/admin (admin only)",
      listStudents: "GET /api/students (token needed)",
      createStudent: "POST /api/students (admin only)",
      deleteStudent: "DELETE /api/students/:id (admin only)"
    }
  });
});

app.use("/api/auth", authRoutes);
app.use("/api/students", studentRoutes);

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
  });
}

startServer();
```

---

## Testing the API

Start the server

```bash
node --watch server.js
```

Register the first user and make it an admin (in a second terminal)

```bash
curl -X POST http://localhost:5000/api/auth/register -H "Content-Type: application/json" -d '{"name":"Admin","email":"admin@example.com","password":"admin123"}'
node make-admin.js admin@example.com
```

On Windows, send the register request with Postman or the PowerShell commands from [Session 11](11-crud-operations-dummy-data.md#testing-your-api).

test-api.js tests every case

```javascript
const BASE = "http://localhost:5000/api";

async function send(method, path, body, token) {
  const headers = { "Content-Type": "application/json" };
  if (token) {
    headers.Authorization = `Bearer ${token}`;
  }

  const res = await fetch(BASE + path, {
    method,
    headers,
    body: body ? JSON.stringify(body) : undefined
  });
  const data = await res.json();
  return { status: res.status, data };
}

function show(label, { status, data }) {
  let info = data.message || "";
  if (data.errors) info += " " + JSON.stringify(data.errors);
  if (data.user) info += ` ${data.user.name} (${data.user.role})`;
  if (data.token) info += " + token";
  if (Array.isArray(data.data)) info += ` ${data.count} students`;
  console.log(`${label.padEnd(36)} ${status} ${info.trim()}`);
}

async function test() {
  // Admin was registered before and promoted with make-admin.js
  const adminLogin = await send("POST", "/auth/login", { email: "admin@example.com", password: "admin123" });
  show("Login admin", adminLogin);
  const adminToken = adminLogin.data.token;

  const reg = await send("POST", "/auth/register", { name: "Sara", email: "sara@example.com", password: "secret123", role: "admin" });
  show("Register Sara (tries role admin)", reg);
  const saraToken = reg.data.token;

  show("Register same email", await send("POST", "/auth/register", { name: "Copy", email: "sara@example.com", password: "secret123" }));
  show("Register short password", await send("POST", "/auth/register", { name: "Tim", email: "tim@example.com", password: "123" }));
  show("Login wrong password", await send("POST", "/auth/login", { email: "sara@example.com", password: "nope" }));
  show("Login unknown email", await send("POST", "/auth/login", { email: "nobody@example.com", password: "secret123" }));
  show("GET /auth/me without token", await send("GET", "/auth/me"));
  show("GET /auth/me with bad token", await send("GET", "/auth/me", null, "abc.def.ghi"));
  show("GET /auth/me as Sara", await send("GET", "/auth/me", null, saraToken));
  show("GET /students as Sara", await send("GET", "/students", null, saraToken));
  show("POST /students as Sara", await send("POST", "/students", { name: "Ali", age: 21, course: "Physics" }, saraToken));
  const created = await send("POST", "/students", { name: "Ali", age: 21, course: "Physics" }, adminToken);
  show("POST /students as admin", created);
  show("GET /auth/admin as Sara", await send("GET", "/auth/admin", null, saraToken));
  show("GET /auth/admin as admin", await send("GET", "/auth/admin", null, adminToken));
  show("DELETE /students/:id as admin", await send("DELETE", "/students/" + created.data.data._id, null, adminToken));
}

test();
```

```bash
node test-api.js
```

Output

```text
Login admin                          200 Admin (admin) + token
Register Sara (tries role admin)     201 Sara (user) + token
Register same email                  409 An account with this email already exists
Register short password              400 Validation failed ["Password must be at least 6 characters"]
Login wrong password                 401 Invalid email or password
Login unknown email                  401 Invalid email or password
GET /auth/me without token           401 Not logged in. Please send a token.
GET /auth/me with bad token          401 Invalid token. Please log in again.
GET /auth/me as Sara                 200 Sara (user)
GET /students as Sara                200 0 students
POST /students as Sara               403 Role "user" is not allowed to do this
POST /students as admin              201
GET /auth/admin as Sara              403 Role "user" is not allowed to do this
GET /auth/admin as admin             200 Welcome Admin, you are an admin
DELETE /students/:id as admin        200 Student deleted successfully
```

![Every auth case and its status code](images/22-authentication-using-jwt/test-run.gif)

We also tested these cases directly

| Case                                         | Result                                           |
| -------------------------------------------- | ------------------------------------------------ |
| Token whose `exp` has passed                 | 401 Your token has expired. Please log in again. |
| Token signed with a guessed secret           | 401 Invalid token. Please log in again.          |
| Valid token, user deactivated afterwards     | 401 This user no longer exists or is deactivated |
| Valid token, user deleted afterwards         | 401 This user no longer exists or is deactivated |
| Login of a deactivated user                  | 403 Your account has been deactivated            |

The test script only works on an empty database (Sara can register only once). Delete the users in Compass to run it again.

With curl (macOS, Linux, Git Bash), copy the token from the login response

```bash
curl http://localhost:5000/api/auth/me -H "Authorization: Bearer YOUR_TOKEN_HERE"
```

In Postman, open the **Authorization** tab, choose **Bearer Token** and paste the token. Postman adds the header for you.

---

## Where the Client Keeps the Token

The server only sends the token. Where the client keeps it matters for security

| Place               | How it is sent                    | Risk                                            |
| ------------------- | --------------------------------- | ----------------------------------------------- |
| Memory (a variable) | Your code adds the header         | Lost on page refresh                            |
| localStorage        | Your code adds the header         | Any script on the page can read it. One XSS bug (Session 21) and the token is stolen |
| httpOnly cookie     | The browser sends it automatically | JavaScript cannot read it. Needs CSRF protection |

For a beginner API used by mobile apps or Postman, the `Authorization` header is the standard. For a website, httpOnly cookies are safer. Whatever you choose, never put a token in a URL like `?token=...`: URLs end up in logs and browser history.

---

## Logging Out

With a JWT, the server stores nothing, so there is nothing to delete on the server. Logging out means **the client throws the token away**.

But a stolen copy of the token keeps working until `exp`. That is why

* Tokens should be short-lived for sensitive apps (`15m` to `1h`). Big apps add a long-lived "refresh token" to get new short tokens.
* `protect` checks that the user is still active, so an admin can block someone right away.
* Changing `JWT_SECRET` makes every token invalid at once (everyone must log in again).

---

## Beginner Mistakes

### Mistake 1

Taking the role from the request body at register.

`role: req.body.role || "user"` lets anyone become an admin. Never trust the client with permissions.

---

### Mistake 2

Using `next` in the Mongoose 9 save hook.

`async function (next) { ... next(); }` makes every registration fail with `TypeError: next is not a function`.

---

### Mistake 3

Putting secrets in the payload.

The payload is only encoded, not encrypted. Anyone can read it on jwt.io.

---

### Mistake 4

`JWT_EXPIRES_IN=3600`.

The string `"3600"` means 3.6 seconds. Write `1h`.

---

### Mistake 5

A weak or committed `JWT_SECRET`.

With the secret, anyone can create an admin token. Use a long random value and keep it in .env.

---

### Mistake 6

Sending the user document after `create()` or `select("+password")`.

It contains the password hash. Pick the fields to send.

---

### Mistake 7

Different messages for "unknown email" and "wrong password".

Attackers learn which emails are registered. Use one message.

---

### Mistake 8

Forgetting the space in `"Bearer "`, or sending only the token.

The header must be exactly `Authorization: Bearer <token>`.

---

### Mistake 9

Putting `authorize()` before `protect`.

`req.user` does not exist yet, so the app crashes with "Cannot read properties of undefined (reading 'role')".

---

### Mistake 10

Using a user's input directly in a query.

`User.findOne({ email: req.body.email })` accepts `{ "$gt": "" }`. Convert to a string first.

---

## Practice Exercises

### Exercise 1

Add `PATCH /api/auth/me` so a logged-in user can change their name. Do not allow changing the role or email here

### Exercise 2

Add `PATCH /api/auth/password`. The body has `currentPassword` and `newPassword`. Check the current one with `comparePassword`, then set `user.password = newPassword` and `await user.save()` so the hook hashes it. Why does `findByIdAndUpdate` not work here? (Session 19)

### Exercise 3

Add an admin route `PATCH /api/auth/users/:id/deactivate` that sets `isActive: false`. Test that the user's old token stops working

### Exercise 4

Add rate limiting to the login route with express-rate-limit (Session 14): at most 5 tries per 15 minutes. This stops password guessing

### Exercise 5

Add a third role, "teacher". Teachers can create students but not delete them. Use `authorize("admin", "teacher")`

### Exercise 6

Set `JWT_EXPIRES_IN=1m`, log in, wait a minute, and call `/api/auth/me`. Which message do you get?

### Exercise 7

Paste one of your tokens into jwt.io. Find your user id and the expiry time. Change one letter of the payload and send the changed token to `/api/auth/me`

---

## Interview Questions

### What is the difference between authentication and authorization

Authentication verifies who you are (401 when it fails). Authorization checks what you are allowed to do (403 when it fails)

### What is JWT

JSON Web Token: a signed token the server gives after login. The client sends it with each request to prove who it is

### What are the three parts of a JWT

Header (algorithm), Payload (claims like the user id and exp), and Signature

### Is the JWT payload encrypted

No. It is only base64url encoded, anyone can read it. The signature only stops changes, it does not hide data

### How does the server know a token was not changed

It recalculates the signature from the header, payload and its secret. If the payload was changed, the signatures do not match and verify throws "invalid signature"

### What does jwt.sign do

It creates a token from a payload, signed with the secret, optionally with an expiry

### What does jwt.verify do

It checks the signature and the expiry, and returns the payload. It throws JsonWebTokenError or TokenExpiredError when the token is bad

### Why do we need JWT_SECRET

It is used to sign and verify tokens. Anyone who knows it can create valid tokens for any user

### What is the difference between sessions and JWT

Sessions store login state on the server and send a session id. JWTs store nothing on the server, the signed token carries the user id

### How do you log out with JWT

The client deletes the token. The server cannot cancel a token, so use short expiry times and check that the user is still active

### Where should a client store a JWT

In an Authorization header from memory for apps, or an httpOnly cookie for websites. localStorage is readable by any script, so an XSS bug can steal it

### Why put only the user id in the token, not the role

The server reads the current role from the database on each request, so a role change works immediately instead of after the old token expires

### Why use the same error message for wrong email and wrong password

So attackers cannot find out which emails have accounts

### What is NoSQL injection

Sending query operators like { "$gt": "" } instead of a value, so the query matches more than intended. Convert input to the expected type

### Why do we hash passwords

So that a stolen database does not reveal the real passwords. Session 23 explains how

---
