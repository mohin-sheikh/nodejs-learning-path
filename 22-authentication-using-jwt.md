## Table of Contents

* [What is Authentication](#what-is-authentication)
* [What is Authorization](#what-is-authorization)
* [What is JWT](#what-is-jwt)
* [How JWT Works](#how-jwt-works)
* [JWT Structure](#jwt-structure)
* [Setting Up the Project](#setting-up-the-project)
* [Creating the User Model](#creating-the-user-model)
* [Register User](#register-user)
* [Login User](#login-user)
* [Creating JWT Token](#creating-jwt-token)
* [Verifying JWT Token](#verifying-jwt-token)
* [Protected Routes](#protected-routes)
* [Complete Authentication API](#complete-authentication-api)
* [Testing the API](#testing-the-api)
* [Practice Exercises](#practice-exercises)
* [Interview Questions](#interview-questions)
* [Summary](#summary)

---

## What is Authentication

Authentication is verifying who someone is

It answers the question "Are you who you say you are?"

Think of it like showing your ID at airport security

```text
You show your passport
Security checks it
Security confirms it is you
You are authenticated
```

In web applications, authentication is usually done with

```text
Email and password
Username and password
Google login
Facebook login
```

Example

```text
User enters email and password
Server checks if they match
If match, user is authenticated
If not match, authentication fails
```

---

## What is Authorization

Authorization is what someone is allowed to do

It answers the question "What can you do?"

Think of it like a hotel room key

```text
Hotel guest has a room key
Key only opens their room
Cannot open other rooms
Cannot access staff areas
```

In web applications, authorization controls

```text
Which pages user can see
Which data user can access
Which actions user can perform
```

Difference between Authentication and Authorization

| Authentication | Authorization |
| -------------- | ------------- |
| Who are you | What can you do |
| Before authorization | After authentication |
| Login with password | Check user role |
| Verifies identity | Verifies permissions |

Example

```text
Authentication -> You logged in as John
Authorization -> John can view his own profile but cannot delete other users
```

---

## What is JWT

JWT stands for JSON Web Token

It is a way to securely send information between parties

Think of it like a stamp or badge

```text
You visit a building
Security gives you a visitor badge
Badge has your name and photo
You wear the badge everywhere
Security checks badge at each door
```

JWT works the same way

```text
User logs in
Server creates a JWT token
Server sends token to user
User sends token with every request
Server verifies token
Server allows access
```

JWT is stateless

Server does not store any session information

All information is inside the token

---

## How JWT Works

Here is the flow

```text
Step 1 - User registers or logs in
                |
                v
Step 2 - Server creates JWT token
                |
                v
Step 3 - Server sends token to user
                |
                v
Step 4 - User saves token (localStorage or cookie)
                |
                v
Step 5 - User sends token with every request
                |
                v
Step 6 - Server verifies token
                |
                v
Step 7 - Server allows or denies access
```

Example request with token

```text
GET /api/profile
Authorization: Bearer eyJhbGciOiJIUzI1NiIs...
```

The token is sent in the Authorization header

---

## JWT Structure

A JWT token has three parts separated by dots

```text
xxxxx.yyyyy.zzzzz
```

Part 1 - Header

```json
{
  "alg": "HS256",
  "typ": "JWT"
}
```

Part 2 - Payload (the data)

```json
{
  "userId": "12345",
  "email": "john@example.com",
  "role": "user",
  "exp": 1234567890
}
```

Part 3 - Signature (verifies token is genuine)

```text
Signature = HMACSHA256(header + "." + payload, secret)
```

The three parts combined look like this

```text
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VySWQiOiIxMjM0NSIsImVtYWlsIjoiam9obkBleGFtcGxlLmNvbSIsInJvbGUiOiJ1c2VyIn0.sFv3Y5KqJF7QhCqZhZVrKxZVakQyZJHvQrLqQ7LqQ7L
```

The token contains information

No need to look up session in database

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

What each package does

```text
express -> Web framework
mongoose -> MongoDB connection
dotenv -> Environment variables
bcryptjs -> Password hashing
jsonwebtoken -> Create and verify JWT tokens
```

Create .env file

```text
PORT=5000
MONGODB_URI=mongodb+srv://yourusername:yourpassword@cluster0.abc123.mongodb.net/
DB_NAME=auth_demo
JWT_SECRET=my_super_secret_key_change_this_in_production
JWT_EXPIRE=7d
```

JWT_SECRET is very important

Keep it secret

Never share it or commit to GitHub

---

## Creating the User Model

Create models/User.js

```javascript
const mongoose = require("mongoose");
const bcrypt = require("bcryptjs");

const userSchema = new mongoose.Schema({
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
    match: [/^\w+([\.-]?\w+)*@\w+([\.-]?\w+)*(\.\w{2,3})+$/, "Please enter a valid email"]
  },
  password: {
    type: String,
    required: [true, "Password is required"],
    minlength: [6, "Password must be at least 6 characters"],
    select: false  // Do not return password by default
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
}, {
  timestamps: true
});

// Encrypt password before saving
userSchema.pre("save", async function(next) {
  // Only hash if password is modified
  if (!this.isModified("password")) {
    return next();
  }
  
  // Generate salt and hash password
  const salt = await bcrypt.genSalt(10);
  this.password = await bcrypt.hash(this.password, salt);
  next();
});

// Method to compare password
userSchema.methods.comparePassword = async function(enteredPassword) {
  return await bcrypt.compare(enteredPassword, this.password);
};

module.exports = mongoose.model("User", userSchema);
```

Important parts explained

```text
password: { select: false } -> Password not returned in queries
pre("save") -> Runs before saving, hashes password
comparePassword -> Method to check if password is correct
```

---

## Register User

Create controllers/authController.js

```javascript
const User = require("../models/User");
const jwt = require("jsonwebtoken");

// Generate JWT token
const generateToken = (id) => {
  return jwt.sign({ id }, process.env.JWT_SECRET, {
    expiresIn: process.env.JWT_EXPIRE || "7d"
  });
};

// Register user
const register = async (req, res) => {
  try {
    const { name, email, password, role } = req.body;
    
    // Check if user already exists
    const existingUser = await User.findOne({ email });
    
    if (existingUser) {
      return res.status(400).json({
        success: false,
        message: "User already exists with this email"
      });
    }
    
    // Create user
    const user = await User.create({
      name,
      email,
      password,
      role: role || "user"
    });
    
    // Generate token
    const token = generateToken(user._id);
    
    res.status(201).json({
      success: true,
      token,
      user: {
        id: user._id,
        name: user.name,
        email: user.email,
        role: user.role
      }
    });
    
  } catch (error) {
    res.status(400).json({
      success: false,
      message: error.message
    });
  }
};

// Login user
const login = async (req, res) => {
  try {
    const { email, password } = req.body;
    
    // Check if email and password are provided
    if (!email || !password) {
      return res.status(400).json({
        success: false,
        message: "Please provide email and password"
      });
    }
    
    // Find user and include password (since select: false)
    const user = await User.findOne({ email }).select("+password");
    
    // Check if user exists
    if (!user) {
      return res.status(401).json({
        success: false,
        message: "Invalid credentials"
      });
    }
    
    // Check if password is correct
    const isPasswordMatch = await user.comparePassword(password);
    
    if (!isPasswordMatch) {
      return res.status(401).json({
        success: false,
        message: "Invalid credentials"
      });
    }
    
    // Check if user is active
    if (!user.isActive) {
      return res.status(401).json({
        success: false,
        message: "Your account has been deactivated"
      });
    }
    
    // Generate token
    const token = generateToken(user._id);
    
    res.status(200).json({
      success: true,
      token,
      user: {
        id: user._id,
        name: user.name,
        email: user.email,
        role: user.role
      }
    });
    
  } catch (error) {
    res.status(500).json({
      success: false,
      message: error.message
    });
  }
};

// Get current logged in user
const getMe = async (req, res) => {
  try {
    // req.user comes from protect middleware
    const user = await User.findById(req.user.id);
    
    res.status(200).json({
      success: true,
      data: user
    });
  } catch (error) {
    res.status(500).json({
      success: false,
      message: error.message
    });
  }
};

module.exports = {
  register,
  login,
  getMe
};
```

---

## Creating JWT Token

The generateToken function creates a JWT

```javascript
const generateToken = (id) => {
  return jwt.sign({ id }, process.env.JWT_SECRET, {
    expiresIn: process.env.JWT_EXPIRE
  });
};
```

What happens inside

```text
jwt.sign() -> Creates a new token
{ id } -> Data to store in token
process.env.JWT_SECRET -> Secret key to sign token
expiresIn -> Token expires after 7 days
```

The token contains the user id

Server can extract user id from token later

---

## Verifying JWT Token

We need middleware to verify tokens

Create middleware/authMiddleware.js

```javascript
const jwt = require("jsonwebtoken");
const User = require("../models/User");

// Protect routes - verify token
const protect = async (req, res, next) => {
  let token;
  
  // Check if token exists in headers
  if (req.headers.authorization && req.headers.authorization.startsWith("Bearer")) {
    token = req.headers.authorization.split(" ")[1];
  }
  
  // Check if token exists
  if (!token) {
    return res.status(401).json({
      success: false,
      message: "Not authorized to access this route. Please login."
    });
  }
  
  try {
    // Verify token
    const decoded = jwt.verify(token, process.env.JWT_SECRET);
    
    // Find user and attach to request
    req.user = await User.findById(decoded.id);
    
    if (!req.user) {
      return res.status(401).json({
        success: false,
        message: "User not found"
      });
    }
    
    next();
  } catch (error) {
    return res.status(401).json({
      success: false,
      message: "Not authorized. Invalid token."
    });
  }
};

// Grant access to specific roles
const authorize = (...roles) => {
  return (req, res, next) => {
    if (!roles.includes(req.user.role)) {
      return res.status(403).json({
        success: false,
        message: `Role ${req.user.role} is not authorized to access this route`
      });
    }
    next();
  };
};

module.exports = { protect, authorize };
```

The protect middleware

```text
Gets token from Authorization header
Verifies token using JWT_SECRET
Finds user from database
Attaches user to req.user
Allows access or returns error
```

---

## Protected Routes

Now let us create routes that require authentication

Create routes/userRoutes.js

```javascript
const express = require("express");
const router = express.Router();

const { register, login, getMe } = require("../controllers/authController");
const { protect, authorize } = require("../middleware/authMiddleware");

// Public routes (no authentication needed)
router.post("/register", register);
router.post("/login", login);

// Protected routes (need authentication)
router.get("/me", protect, getMe);

// Admin only routes
router.get("/admin", protect, authorize("admin"), (req, res) => {
  res.json({
    success: true,
    message: "Welcome admin! You have access to admin panel."
  });
});

module.exports = router;
```

Now create a protected student route

Create routes/studentRoutes.js

```javascript
const express = require("express");
const router = express.Router();
const { protect, authorize } = require("../middleware/authMiddleware");
const Student = require("../models/Student");

// All student routes require authentication
router.use(protect);

// Get all students (any logged in user)
router.get("/", async (req, res) => {
  try {
    const students = await Student.find();
    res.json({ success: true, data: students });
  } catch (error) {
    res.status(500).json({ success: false, message: error.message });
  }
});

// Create student (only admin)
router.post("/", authorize("admin"), async (req, res) => {
  try {
    const student = await Student.create(req.body);
    res.status(201).json({ success: true, data: student });
  } catch (error) {
    res.status(400).json({ success: false, message: error.message });
  }
});

// Delete student (only admin)
router.delete("/:id", authorize("admin"), async (req, res) => {
  try {
    await Student.findByIdAndDelete(req.params.id);
    res.json({ success: true, message: "Student deleted" });
  } catch (error) {
    res.status(500).json({ success: false, message: error.message });
  }
});

module.exports = router;
```

---

## Complete Authentication API

Create server.js

```javascript
require("dotenv").config();
const express = require("express");
const mongoose = require("mongoose");

const userRoutes = require("./routes/userRoutes");
const studentRoutes = require("./routes/studentRoutes");

const app = express();
app.use(express.json());

// Routes
app.use("/api/auth", userRoutes);
app.use("/api/students", studentRoutes);

// Home route
app.get("/", (req, res) => {
  res.json({
    message: "JWT Authentication API",
    endpoints: {
      register: "POST /api/auth/register",
      login: "POST /api/auth/login",
      getProfile: "GET /api/auth/me (需要登录)",
      getStudents: "GET /api/students (需要登录)",
      createStudent: "POST /api/students (需要管理员)",
      deleteStudent: "DELETE /api/students/:id (需要管理员)"
    }
  });
});

// Connect to MongoDB
mongoose.connect(`${process.env.MONGODB_URI}/${process.env.DB_NAME}`)
  .then(() => {
    console.log("Connected to MongoDB");
    const PORT = process.env.PORT || 5000;
    app.listen(PORT, () => {
      console.log(`Server running on port ${PORT}`);
    });
  })
  .catch(err => {
    console.error("MongoDB connection failed", err);
    process.exit(1);
  });
```

---

## Testing the API

Start the server

```bash
node server.js
```

### Step 1 - Register a user

```bash
curl -X POST http://localhost:5000/api/auth/register \
  -H "Content-Type: application/json" \
  -d '{
    "name": "John Doe",
    "email": "john@example.com",
    "password": "123456",
    "role": "admin"
  }'
```

Response

```json
{
  "success": true,
  "token": "eyJhbGciOiJIUzI1NiIs...",
  "user": {
    "id": "507f1f77bcf86cd799439011",
    "name": "John Doe",
    "email": "john@example.com",
    "role": "admin"
  }
}
```

Save the token

### Step 2 - Login

```bash
curl -X POST http://localhost:5000/api/auth/login \
  -H "Content-Type: application/json" \
  -d '{
    "email": "john@example.com",
    "password": "123456"
  }'
```

Response includes a new token

### Step 3 - Get profile (protected route)

```bash
curl http://localhost:5000/api/auth/me \
  -H "Authorization: Bearer YOUR_TOKEN_HERE"
```

### Step 4 - Get students (protected route)

```bash
curl http://localhost:5000/api/students \
  -H "Authorization: Bearer YOUR_TOKEN_HERE"
```

### Step 5 - Create student (admin only)

```bash
curl -X POST http://localhost:5000/api/students \
  -H "Authorization: Bearer YOUR_TOKEN_HERE" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Test Student",
    "age": 20,
    "course": "Computer Science",
    "email": "test@example.com"
  }'
```

### Step 6 - Try without token

```bash
curl http://localhost:5000/api/students
```

Response

```json
{
  "success": false,
  "message": "Not authorized to access this route. Please login."
}
```

---

## Practice Exercises

### Exercise 1

Add a password reset feature

Create route POST /api/auth/forgot-password

### Exercise 2

Add logout feature

Invalidate token on server side

### Exercise 3

Add refresh token feature

Create long term refresh token and short term access token

### Exercise 4

Add rate limiting to login route

Prevent brute force attacks

### Exercise 5

Create a profile update route

Users can update their name and password

---

## Interview Questions

### What is JWT

JWT stands for JSON Web Token, used for secure information transmission between parties

### What are the three parts of a JWT

Header, Payload, and Signature

### What is the difference between authentication and authorization

Authentication verifies identity, authorization verifies permissions

### Why do we hash passwords

To protect user passwords if database is compromised

### What is bcrypt

bcrypt is a library that hashes passwords with salt for security

### What does jwt.sign do

It creates a new JWT token with the provided payload

### What does jwt.verify do

It verifies that a token is valid and not tampered with

### Why do we need JWT_SECRET

It is used to sign and verify tokens, keeping them secure

### Where should users store JWT tokens

localStorage or httpOnly cookies

---

## Summary

In this session, you learned

* What authentication and authorization are
* What JWT is and how it works
* JWT structure (Header, Payload, Signature)
* How to register users with hashed passwords
* How to login and generate JWT tokens
* How to create protected routes
* How to verify tokens with middleware
* How to implement role-based authorization
* How to test authenticated APIs

You have built a complete authentication system

---