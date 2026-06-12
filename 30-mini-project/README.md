## Table of Contents

* [Project Overview](#project-overview)
* [Project Requirements](#project-requirements)
* [Project Structure](#project-structure)
* [Step 1 - Setup and Configuration](#step-1---setup-and-configuration)
* [Step 2 - Database Models](#step-2---database-models)
* [Step 3 - Utility Files](#step-3---utility-files)
* [Step 4 - Middleware](#step-4---middleware)
* [Step 5 - Services](#step-5---services)
* [Step 6 - Controllers](#step-6---controllers)
* [Step 7 - Routes](#step-7---routes)
* [Step 8 - Main App](#step-8---main-app)
* [Step 9 - Testing](#step-9---testing)
* [Running the Project](#running-the-project)
* [API Endpoints Summary](#api-endpoints-summary)
* [Practice Exercises](#practice-exercises)
* [Summary](#summary)

---

## Project Overview

We will build a Task Management System

This is a complete backend API

Users can

```text
Register and login
Create tasks
View their tasks
Update tasks
Delete tasks
Mark tasks as complete
Filter tasks by status
Upload profile picture
Reset password
```

This project uses everything from Sessions 1 to 29

Technologies used

```text
Node.js
Express.js
MongoDB with Mongoose
JWT for authentication
bcryptjs for password hashing
Multer for file upload
Jest for testing
```

---

## Project Requirements

Functional Requirements

```text
User can register with name, email, password
User can login and receive JWT token
User can create a task (title, description, dueDate)
User can view all their tasks
User can view a single task
User can update a task
User can delete a task
User can mark task as complete/incomplete
User can filter tasks by status (pending/completed)
User can upload profile picture
User can reset password
Only authenticated users can access tasks
Users can only see their own tasks
```

Technical Requirements

```text
MVC architecture
Environment variables
Error handling
Input validation
Logging
API documentation with Swagger
Unit and integration tests
Proper HTTP status codes
Security best practices
```

---

## Project Structure

```text
task-management-api/
│
├── src/
│   ├── config/
│   │   └── database.js
│   │
│   ├── constants/
│   │   ├── statusCodes.js
│   │   └── errorMessages.js
│   │
│   ├── controllers/
│   │   ├── authController.js
│   │   ├── taskController.js
│   │   └── userController.js
│   │
│   ├── middleware/
│   │   ├── authMiddleware.js
│   │   ├── errorMiddleware.js
│   │   ├── uploadMiddleware.js
│   │   └── validationMiddleware.js
│   │
│   ├── models/
│   │   ├── User.js
│   │   └── Task.js
│   │
│   ├── routes/
│   │   ├── authRoutes.js
│   │   ├── taskRoutes.js
│   │   ├── userRoutes.js
│   │   └── index.js
│   │
│   ├── services/
│   │   └── emailService.js
│   │
│   ├── utils/
│   │   ├── AppError.js
│   │   ├── catchAsync.js
│   │   ├── responseHelper.js
│   │   └── validators.js
│   │
│   ├── validation/
│   │   ├── authValidation.js
│   │   └── taskValidation.js
│   │
│   └── app.js
│
├── tests/
│   ├── unit/
│   │   ├── auth.test.js
│   │   └── task.test.js
│   └── integration/
│       └── api.test.js
│
├── uploads/
│   └── profiles/
│
├── logs/
│
├── .env
├── .env.example
├── .gitignore
├── package.json
└── server.js
```

---

## Step 1 - Setup and Configuration

Create project

```bash
mkdir task-management-api
cd task-management-api
npm init -y
```

Install dependencies

```bash
npm install express mongoose bcryptjs jsonwebtoken multer
npm install dotenv cors helmet morgan express-rate-limit
npm install validator
```

Install dev dependencies

```bash
npm install --save-dev jest supertest nodemon
```

package.json scripts

```json
{
  "scripts": {
    "start": "node server.js",
    "dev": "nodemon server.js",
    "test": "jest",
    "test:watch": "jest --watch",
    "test:coverage": "jest --coverage"
  }
}
```

Create .env file

```text
NODE_ENV=development
PORT=5000

MONGODB_URI=mongodb+srv://username:password@cluster0.abc123.mongodb.net/
DB_NAME=task_manager

JWT_SECRET=my_super_secret_jwt_key_change_in_production
JWT_EXPIRE=7d

EMAIL_HOST=smtp.gmail.com
EMAIL_PORT=587
EMAIL_USER=your-email@gmail.com
EMAIL_PASS=your-app-password
```

Create .env.example

```text
NODE_ENV=development
PORT=5000
MONGODB_URI=your_mongodb_uri
DB_NAME=task_manager
JWT_SECRET=your_jwt_secret
JWT_EXPIRE=7d
EMAIL_HOST=smtp.gmail.com
EMAIL_PORT=587
EMAIL_USER=your-email@gmail.com
EMAIL_PASS=your-app-password
```

Create .gitignore

```text
node_modules/
.env
uploads/
logs/
coverage/
.DS_Store
```

---

## Step 2 - Database Models

src/config/database.js

```javascript
const mongoose = require("mongoose");

const connectDB = async () => {
  try {
    const conn = await mongoose.connect(`${process.env.MONGODB_URI}/${process.env.DB_NAME}`);
    console.log(`MongoDB Connected: ${conn.connection.host}`);
  } catch (error) {
    console.error(`Error: ${error.message}`);
    process.exit(1);
  }
};

module.exports = connectDB;
```

src/models/User.js

```javascript
const mongoose = require("mongoose");
const bcrypt = require("bcryptjs");
const validator = require("validator");

const userSchema = new mongoose.Schema({
  name: {
    type: String,
    required: [true, "Name is required"],
    trim: true,
    minlength: [2, "Name must be at least 2 characters"],
    maxlength: [50, "Name cannot exceed 50 characters"]
  },
  email: {
    type: String,
    required: [true, "Email is required"],
    unique: true,
    lowercase: true,
    trim: true,
    validate: [validator.isEmail, "Please provide a valid email"]
  },
  password: {
    type: String,
    required: [true, "Password is required"],
    minlength: [6, "Password must be at least 6 characters"],
    select: false
  },
  profilePicture: {
    type: String,
    default: "default.jpg"
  },
  role: {
    type: String,
    enum: ["user", "admin"],
    default: "user"
  },
  isActive: {
    type: Boolean,
    default: true
  },
  resetPasswordToken: String,
  resetPasswordExpire: Date
}, {
  timestamps: true
});

// Hash password before saving
userSchema.pre("save", async function(next) {
  if (!this.isModified("password")) {
    return next();
  }
  
  const salt = await bcrypt.genSalt(10);
  this.password = await bcrypt.hash(this.password, salt);
  next();
});

// Compare password method
userSchema.methods.comparePassword = async function(enteredPassword) {
  return await bcrypt.compare(enteredPassword, this.password);
};

module.exports = mongoose.model("User", userSchema);
```

src/models/Task.js

```javascript
const mongoose = require("mongoose");

const taskSchema = new mongoose.Schema({
  title: {
    type: String,
    required: [true, "Title is required"],
    trim: true,
    maxlength: [100, "Title cannot exceed 100 characters"]
  },
  description: {
    type: String,
    trim: true,
    maxlength: [1000, "Description cannot exceed 1000 characters"]
  },
  status: {
    type: String,
    enum: ["pending", "in-progress", "completed"],
    default: "pending"
  },
  dueDate: {
    type: Date
  },
  user: {
    type: mongoose.Schema.Types.ObjectId,
    ref: "User",
    required: true
  }
}, {
  timestamps: true
});

// Index for faster queries
taskSchema.index({ user: 1, status: 1 });
taskSchema.index({ user: 1, createdAt: -1 });

module.exports = mongoose.model("Task", taskSchema);
```

---

## Step 3 - Utility Files

src/utils/AppError.js

```javascript
class AppError extends Error {
  constructor(message, statusCode) {
    super(message);
    this.statusCode = statusCode;
    this.status = `${statusCode}`.startsWith("4") ? "fail" : "error";
    this.isOperational = true;
    
    Error.captureStackTrace(this, this.constructor);
  }
}

module.exports = AppError;
```

src/utils/catchAsync.js

```javascript
const catchAsync = (fn) => {
  return (req, res, next) => {
    fn(req, res, next).catch(next);
  };
};

module.exports = catchAsync;
```

src/utils/responseHelper.js

```javascript
class ResponseHelper {
  static success(res, data, message = "Success", statusCode = 200) {
    return res.status(statusCode).json({
      success: true,
      message,
      data
    });
  }
  
  static error(res, message, statusCode = 400) {
    return res.status(statusCode).json({
      success: false,
      message
    });
  }
  
  static created(res, data, message = "Resource created successfully") {
    return this.success(res, data, message, 201);
  }
  
  static notFound(res, message = "Resource not found") {
    return this.error(res, message, 404);
  }
  
  static badRequest(res, message = "Bad request") {
    return this.error(res, message, 400);
  }
  
  static unauthorized(res, message = "Unauthorized") {
    return this.error(res, message, 401);
  }
  
  static forbidden(res, message = "Forbidden") {
    return this.error(res, message, 403);
  }
}

module.exports = ResponseHelper;
```

src/utils/validators.js

```javascript
const validateEmail = (email) => {
  const regex = /^\w+([\.-]?\w+)*@\w+([\.-]?\w+)*(\.\w{2,3})+$/;
  return regex.test(email);
};

const validatePassword = (password) => {
  return password.length >= 6;
};

const validateObjectId = (id) => {
  const mongoose = require("mongoose");
  return mongoose.Types.ObjectId.isValid(id);
};

const validateDate = (date) => {
  const parsedDate = new Date(date);
  return !isNaN(parsedDate.getTime());
};

module.exports = {
  validateEmail,
  validatePassword,
  validateObjectId,
  validateDate
};
```

src/constants/statusCodes.js

```javascript
const HTTP_STATUS = {
  OK: 200,
  CREATED: 201,
  BAD_REQUEST: 400,
  UNAUTHORIZED: 401,
  FORBIDDEN: 403,
  NOT_FOUND: 404,
  CONFLICT: 409,
  INTERNAL_SERVER_ERROR: 500
};

module.exports = HTTP_STATUS;
```

src/constants/errorMessages.js

```javascript
const ERROR_MESSAGES = {
  INVALID_CREDENTIALS: "Invalid email or password",
  UNAUTHORIZED: "You are not logged in",
  TOKEN_EXPIRED: "Token expired. Please login again",
  INVALID_TOKEN: "Invalid token",
  USER_NOT_FOUND: "User not found",
  EMAIL_EXISTS: "Email already exists",
  TASK_NOT_FOUND: "Task not found",
  MISSING_FIELDS: "Missing required fields",
  INVALID_EMAIL: "Please provide a valid email",
  WEAK_PASSWORD: "Password must be at least 6 characters",
  INTERNAL_ERROR: "Internal server error"
};

module.exports = ERROR_MESSAGES;
```

---

## Step 4 - Middleware

src/middleware/authMiddleware.js

```javascript
const jwt = require("jsonwebtoken");
const User = require("../models/User");
const AppError = require("../utils/AppError");
const catchAsync = require("../utils/catchAsync");

const protect = catchAsync(async (req, res, next) => {
  let token;
  
  if (req.headers.authorization && req.headers.authorization.startsWith("Bearer")) {
    token = req.headers.authorization.split(" ")[1];
  }
  
  if (!token) {
    return next(new AppError("Not authorized. Please login.", 401));
  }
  
  try {
    const decoded = jwt.verify(token, process.env.JWT_SECRET);
    req.user = await User.findById(decoded.id);
    
    if (!req.user) {
      return next(new AppError("User not found", 401));
    }
    
    next();
  } catch (error) {
    return next(new AppError("Invalid token. Please login again.", 401));
  }
});

const authorize = (...roles) => {
  return (req, res, next) => {
    if (!roles.includes(req.user.role)) {
      return next(new AppError(`Role ${req.user.role} is not authorized`, 403));
    }
    next();
  };
};

module.exports = { protect, authorize };
```

src/middleware/uploadMiddleware.js

```javascript
const multer = require("multer");
const path = require("path");
const AppError = require("../utils/AppError");

const storage = multer.diskStorage({
  destination: (req, file, cb) => {
    cb(null, "uploads/profiles/");
  },
  filename: (req, file, cb) => {
    const uniqueSuffix = Date.now() + "-" + Math.round(Math.random() * 1E9);
    cb(null, `user-${req.user.id}-${uniqueSuffix}${path.extname(file.originalname)}`);
  }
});

const fileFilter = (req, file, cb) => {
  const allowedTypes = ["image/jpeg", "image/png", "image/jpg"];
  
  if (allowedTypes.includes(file.mimetype)) {
    cb(null, true);
  } else {
    cb(new AppError("Only JPEG and PNG images are allowed", 400), false);
  }
};

const upload = multer({
  storage: storage,
  fileFilter: fileFilter,
  limits: {
    fileSize: 1024 * 1024 * 2 // 2MB
  }
});

module.exports = upload;
```

src/middleware/validationMiddleware.js

```javascript
const { body, validationResult } = require("express-validator");

const validate = (req, res, next) => {
  const errors = validationResult(req);
  if (!errors.isEmpty()) {
    return res.status(400).json({
      success: false,
      errors: errors.array().map(err => err.msg)
    });
  }
  next();
};

const registerValidation = [
  body("name").notEmpty().withMessage("Name is required"),
  body("email").isEmail().withMessage("Valid email is required"),
  body("password").isLength({ min: 6 }).withMessage("Password must be at least 6 characters"),
  validate
];

const loginValidation = [
  body("email").isEmail().withMessage("Valid email is required"),
  body("password").notEmpty().withMessage("Password is required"),
  validate
];

const taskValidation = [
  body("title").notEmpty().withMessage("Title is required"),
  body("status").optional().isIn(["pending", "in-progress", "completed"]),
  body("dueDate").optional().isISO8601().withMessage("Invalid date format"),
  validate
];

module.exports = {
  registerValidation,
  loginValidation,
  taskValidation
};
```

src/middleware/errorMiddleware.js

```javascript
const AppError = require("../utils/AppError");

const handleCastErrorDB = (err) => {
  return new AppError(`Invalid ${err.path}: ${err.value}`, 400);
};

const handleDuplicateFieldsDB = (err) => {
  const field = Object.keys(err.keyPattern)[0];
  return new AppError(`${field} already exists. Please use another value.`, 400);
};

const handleValidationErrorDB = (err) => {
  const errors = Object.values(err.errors).map(el => el.message);
  return new AppError(errors.join(". "), 400);
};

const sendErrorDev = (err, res) => {
  res.status(err.statusCode).json({
    success: false,
    message: err.message,
    error: err,
    stack: err.stack
  });
};

const sendErrorProd = (err, res) => {
  if (err.isOperational) {
    res.status(err.statusCode).json({
      success: false,
      message: err.message
    });
  } else {
    console.error("ERROR 💥", err);
    res.status(500).json({
      success: false,
      message: "Something went wrong"
    });
  }
};

module.exports = (err, req, res, next) => {
  err.statusCode = err.statusCode || 500;
  err.status = err.status || "error";
  
  if (process.env.NODE_ENV === "development") {
    sendErrorDev(err, res);
  } else {
    let error = { ...err };
    error.message = err.message;
    
    if (error.name === "CastError") error = handleCastErrorDB(error);
    if (error.code === 11000) error = handleDuplicateFieldsDB(error);
    if (error.name === "ValidationError") error = handleValidationErrorDB(error);
    
    sendErrorProd(error, res);
  }
};
```

---

## Step 5 - Services

src/services/emailService.js

```javascript
const nodemailer = require("nodemailer");

const sendEmail = async (options) => {
  const transporter = nodemailer.createTransport({
    host: process.env.EMAIL_HOST,
    port: process.env.EMAIL_PORT,
    auth: {
      user: process.env.EMAIL_USER,
      pass: process.env.EMAIL_PASS
    }
  });
  
  const mailOptions = {
    from: `Task Manager <${process.env.EMAIL_USER}>`,
    to: options.email,
    subject: options.subject,
    text: options.message
  };
  
  await transporter.sendMail(mailOptions);
};

module.exports = sendEmail;
```

---

## Step 6 - Controllers

src/controllers/authController.js

```javascript
const jwt = require("jsonwebtoken");
const crypto = require("crypto");
const User = require("../models/User");
const AppError = require("../utils/AppError");
const catchAsync = require("../utils/catchAsync");
const ResponseHelper = require("../utils/responseHelper");
const sendEmail = require("../services/emailService");

const generateToken = (id) => {
  return jwt.sign({ id }, process.env.JWT_SECRET, {
    expiresIn: process.env.JWT_EXPIRE
  });
};

const register = catchAsync(async (req, res, next) => {
  const { name, email, password } = req.body;
  
  const existingUser = await User.findOne({ email });
  if (existingUser) {
    return next(new AppError("Email already exists", 400));
  }
  
  const user = await User.create({ name, email, password });
  
  const token = generateToken(user._id);
  
  ResponseHelper.created(res, {
    id: user._id,
    name: user.name,
    email: user.email,
    token
  }, "User registered successfully");
});

const login = catchAsync(async (req, res, next) => {
  const { email, password } = req.body;
  
  const user = await User.findOne({ email }).select("+password");
  
  if (!user || !(await user.comparePassword(password))) {
    return next(new AppError("Invalid email or password", 401));
  }
  
  const token = generateToken(user._id);
  
  ResponseHelper.success(res, {
    id: user._id,
    name: user.name,
    email: user.email,
    token
  }, "Login successful");
});

const getMe = catchAsync(async (req, res, next) => {
  const user = await User.findById(req.user.id);
  
  ResponseHelper.success(res, user, "User profile retrieved");
});

const updateProfile = catchAsync(async (req, res, next) => {
  const { name, email } = req.body;
  
  const user = await User.findByIdAndUpdate(
    req.user.id,
    { name, email },
    { new: true, runValidators: true }
  );
  
  ResponseHelper.success(res, user, "Profile updated successfully");
});

const forgotPassword = catchAsync(async (req, res, next) => {
  const user = await User.findOne({ email: req.body.email });
  
  if (!user) {
    return next(new AppError("User not found", 404));
  }
  
  const resetToken = crypto.randomBytes(32).toString("hex");
  user.resetPasswordToken = crypto.createHash("sha256").update(resetToken).digest("hex");
  user.resetPasswordExpire = Date.now() + 10 * 60 * 1000; // 10 minutes
  await user.save({ validateBeforeSave: false });
  
  const resetUrl = `${req.protocol}://${req.get("host")}/api/auth/reset-password/${resetToken}`;
  
  await sendEmail({
    email: user.email,
    subject: "Password Reset",
    message: `Click here to reset your password: ${resetUrl}`
  });
  
  ResponseHelper.success(res, null, "Password reset email sent");
});

const resetPassword = catchAsync(async (req, res, next) => {
  const resetPasswordToken = crypto.createHash("sha256").update(req.params.token).digest("hex");
  
  const user = await User.findOne({
    resetPasswordToken,
    resetPasswordExpire: { $gt: Date.now() }
  });
  
  if (!user) {
    return next(new AppError("Invalid or expired token", 400));
  }
  
  user.password = req.body.password;
  user.resetPasswordToken = undefined;
  user.resetPasswordExpire = undefined;
  await user.save();
  
  const token = generateToken(user._id);
  
  ResponseHelper.success(res, { token }, "Password reset successful");
});

module.exports = {
  register,
  login,
  getMe,
  updateProfile,
  forgotPassword,
  resetPassword
};
```

src/controllers/taskController.js

```javascript
const Task = require("../models/Task");
const AppError = require("../utils/AppError");
const catchAsync = require("../utils/catchAsync");
const ResponseHelper = require("../utils/responseHelper");

const createTask = catchAsync(async (req, res, next) => {
  const { title, description, dueDate, status } = req.body;
  
  const task = await Task.create({
    title,
    description,
    dueDate,
    status,
    user: req.user.id
  });
  
  ResponseHelper.created(res, task, "Task created successfully");
});

const getAllTasks = catchAsync(async (req, res, next) => {
  const { status, page = 1, limit = 10 } = req.query;
  
  const query = { user: req.user.id };
  if (status) {
    query.status = status;
  }
  
  const tasks = await Task.find(query)
    .sort("-createdAt")
    .limit(limit * 1)
    .skip((page - 1) * limit);
  
  const total = await Task.countDocuments(query);
  
  ResponseHelper.success(res, {
    tasks,
    total,
    page: parseInt(page),
    pages: Math.ceil(total / limit)
  }, "Tasks retrieved successfully");
});

const getTaskById = catchAsync(async (req, res, next) => {
  const task = await Task.findOne({
    _id: req.params.id,
    user: req.user.id
  });
  
  if (!task) {
    return next(new AppError("Task not found", 404));
  }
  
  ResponseHelper.success(res, task, "Task retrieved successfully");
});

const updateTask = catchAsync(async (req, res, next) => {
  const { title, description, dueDate, status } = req.body;
  
  const task = await Task.findOneAndUpdate(
    { _id: req.params.id, user: req.user.id },
    { title, description, dueDate, status },
    { new: true, runValidators: true }
  );
  
  if (!task) {
    return next(new AppError("Task not found", 404));
  }
  
  ResponseHelper.success(res, task, "Task updated successfully");
});

const deleteTask = catchAsync(async (req, res, next) => {
  const task = await Task.findOneAndDelete({
    _id: req.params.id,
    user: req.user.id
  });
  
  if (!task) {
    return next(new AppError("Task not found", 404));
  }
  
  ResponseHelper.success(res, null, "Task deleted successfully");
});

const updateTaskStatus = catchAsync(async (req, res, next) => {
  const { status } = req.body;
  
  const task = await Task.findOneAndUpdate(
    { _id: req.params.id, user: req.user.id },
    { status },
    { new: true }
  );
  
  if (!task) {
    return next(new AppError("Task not found", 404));
  }
  
  ResponseHelper.success(res, task, "Task status updated");
});

const getTaskStats = catchAsync(async (req, res, next) => {
  const stats = await Task.aggregate([
    { $match: { user: req.user._id } },
    { $group: {
      _id: "$status",
      count: { $sum: 1 }
    }}
  ]);
  
  const result = {
    total: 0,
    pending: 0,
    "in-progress": 0,
    completed: 0
  };
  
  stats.forEach(stat => {
    result[stat._id] = stat.count;
    result.total += stat.count;
  });
  
  ResponseHelper.success(res, result, "Task statistics retrieved");
});

module.exports = {
  createTask,
  getAllTasks,
  getTaskById,
  updateTask,
  deleteTask,
  updateTaskStatus,
  getTaskStats
};
```

src/controllers/userController.js

```javascript
const User = require("../models/User");
const AppError = require("../utils/AppError");
const catchAsync = require("../utils/catchAsync");
const ResponseHelper = require("../utils/responseHelper");
const upload = require("../middleware/uploadMiddleware");

const uploadProfilePicture = catchAsync(async (req, res, next) => {
  upload.single("profilePicture")(req, res, async (err) => {
    if (err) {
      return next(new AppError(err.message, 400));
    }
    
    if (!req.file) {
      return next(new AppError("Please upload a file", 400));
    }
    
    const user = await User.findByIdAndUpdate(
      req.user.id,
      { profilePicture: req.file.filename },
      { new: true }
    );
    
    ResponseHelper.success(res, user, "Profile picture uploaded");
  });
});

const deleteAccount = catchAsync(async (req, res, next) => {
  await User.findByIdAndDelete(req.user.id);
  
  ResponseHelper.success(res, null, "Account deleted successfully");
});

module.exports = {
  uploadProfilePicture,
  deleteAccount
};
```

---

## Step 7 - Routes

src/routes/authRoutes.js

```javascript
const express = require("express");
const router = express.Router();
const {
  register,
  login,
  getMe,
  updateProfile,
  forgotPassword,
  resetPassword
} = require("../controllers/authController");
const { protect } = require("../middleware/authMiddleware");
const { registerValidation, loginValidation } = require("../middleware/validationMiddleware");

router.post("/register", registerValidation, register);
router.post("/login", loginValidation, login);
router.post("/forgot-password", forgotPassword);
router.post("/reset-password/:token", resetPassword);

router.use(protect);
router.get("/me", getMe);
router.put("/profile", updateProfile);

module.exports = router;
```

src/routes/taskRoutes.js

```javascript
const express = require("express");
const router = express.Router();
const {
  createTask,
  getAllTasks,
  getTaskById,
  updateTask,
  deleteTask,
  updateTaskStatus,
  getTaskStats
} = require("../controllers/taskController");
const { protect } = require("../middleware/authMiddleware");
const { taskValidation } = require("../middleware/validationMiddleware");

router.use(protect);

router.route("/")
  .get(getAllTasks)
  .post(taskValidation, createTask);

router.get("/stats", getTaskStats);

router.route("/:id")
  .get(getTaskById)
  .put(taskValidation, updateTask)
  .delete(deleteTask);

router.patch("/:id/status", updateTaskStatus);

module.exports = router;
```

src/routes/userRoutes.js

```javascript
const express = require("express");
const router = express.Router();
const {
  uploadProfilePicture,
  deleteAccount
} = require("../controllers/userController");
const { protect } = require("../middleware/authMiddleware");

router.use(protect);

router.post("/upload-profile", uploadProfilePicture);
router.delete("/account", deleteAccount);

module.exports = router;
```

src/routes/index.js

```javascript
const express = require("express");
const router = express.Router();
const authRoutes = require("./authRoutes");
const taskRoutes = require("./taskRoutes");
const userRoutes = require("./userRoutes");

router.use("/auth", authRoutes);
router.use("/tasks", taskRoutes);
router.use("/users", userRoutes);

module.exports = router;
```

---

## Step 8 - Main App

src/app.js

```javascript
const express = require("express");
const cors = require("cors");
const helmet = require("helmet");
const morgan = require("morgan");
const rateLimit = require("express-rate-limit");
const path = require("path");
const fs = require("fs");

const routes = require("./routes");
const errorMiddleware = require("./middleware/errorMiddleware");
const AppError = require("./utils/AppError");

const app = express();

// Create uploads folder if not exists
const uploadDir = "uploads/profiles";
if (!fs.existsSync(uploadDir)) {
  fs.mkdirSync(uploadDir, { recursive: true });
}

// Security middleware
app.use(helmet());
app.use(cors());

// Rate limiting
const limiter = rateLimit({
  windowMs: 15 * 60 * 1000,
  max: 100,
  message: "Too many requests from this IP"
});
app.use("/api", limiter);

// Body parsing middleware
app.use(express.json());
app.use(express.urlencoded({ extended: true }));

// Logging
if (process.env.NODE_ENV === "development") {
  app.use(morgan("dev"));
}

// Static files
app.use("/uploads", express.static("uploads"));

// Routes
app.use("/api", routes);

// Health check
app.get("/health", (req, res) => {
  res.status(200).json({ status: "OK", timestamp: new Date() });
});

// 404 handler
app.all("*", (req, res, next) => {
  next(new AppError(`Route ${req.originalUrl} not found`, 404));
});

// Global error handler
app.use(errorMiddleware);

module.exports = app;
```

server.js

```javascript
const dotenv = require("dotenv");
dotenv.config();

const app = require("./src/app");
const connectDB = require("./src/config/database");

process.on("uncaughtException", (err) => {
  console.log("UNCAUGHT EXCEPTION! Shutting down...");
  console.log(err.name, err.message);
  process.exit(1);
});

connectDB();

const PORT = process.env.PORT || 5000;
const server = app.listen(PORT, () => {
  console.log(`Server running on port ${PORT}`);
  console.log(`Environment: ${process.env.NODE_ENV}`);
});

process.on("unhandledRejection", (err) => {
  console.log("UNHANDLED REJECTION! Shutting down...");
  console.log(err.name, err.message);
  server.close(() => {
    process.exit(1);
  });
});
```

---

## Step 9 - Testing

tests/unit/auth.test.js

```javascript
const request = require("supertest");
const app = require("../../src/app");

describe("Auth API Tests", () => {
  describe("POST /api/auth/register", () => {
    test("should register a new user", async () => {
      const response = await request(app)
        .post("/api/auth/register")
        .send({
          name: "Test User",
          email: "test@example.com",
          password: "123456"
        });
      
      expect(response.status).toBe(201);
      expect(response.body.success).toBe(true);
      expect(response.body.data).toHaveProperty("token");
    });
    
    test("should return error for duplicate email", async () => {
      await request(app)
        .post("/api/auth/register")
        .send({
          name: "User One",
          email: "duplicate@test.com",
          password: "123456"
        });
      
      const response = await request(app)
        .post("/api/auth/register")
        .send({
          name: "User Two",
          email: "duplicate@test.com",
          password: "123456"
        });
      
      expect(response.status).toBe(400);
      expect(response.body.success).toBe(false);
    });
  });
  
  describe("POST /api/auth/login", () => {
    test("should login existing user", async () => {
      await request(app)
        .post("/api/auth/register")
        .send({
          name: "Login User",
          email: "login@test.com",
          password: "123456"
        });
      
      const response = await request(app)
        .post("/api/auth/login")
        .send({
          email: "login@test.com",
          password: "123456"
        });
      
      expect(response.status).toBe(200);
      expect(response.body.success).toBe(true);
      expect(response.body.data).toHaveProperty("token");
    });
    
    test("should return error for invalid credentials", async () => {
      const response = await request(app)
        .post("/api/auth/login")
        .send({
          email: "nonexistent@test.com",
          password: "wrongpass"
        });
      
      expect(response.status).toBe(401);
      expect(response.body.success).toBe(false);
    });
  });
});
```

---

## Running the Project

Start the server

```bash
npm run dev
```

Run tests

```bash
npm test
```

---

## API Endpoints Summary

Authentication

```text
POST   /api/auth/register      - Register new user
POST   /api/auth/login         - Login user
GET    /api/auth/me            - Get current user
PUT    /api/auth/profile       - Update profile
POST   /api/auth/forgot-password - Forgot password
POST   /api/auth/reset-password/:token - Reset password
```

Tasks

```text
GET    /api/tasks              - Get all tasks
POST   /api/tasks              - Create task
GET    /api/tasks/stats        - Get task statistics
GET    /api/tasks/:id          - Get task by id
PUT    /api/tasks/:id          - Update task
DELETE /api/tasks/:id          - Delete task
PATCH  /api/tasks/:id/status   - Update task status
```

Users

```text
POST   /api/users/upload-profile - Upload profile picture
DELETE /api/users/account        - Delete account
```

---

## Practice Exercises

### Exercise 1

Add category to tasks

Each task can belong to a category

### Exercise 2

Add due date reminders

Send email notification one day before due date

### Exercise 3

Add task sharing

Users can share tasks with other users

### Exercise 4

Add task comments

Users can comment on tasks

### Exercise 5

Deploy the project

Deploy to Render, Railway, or Vercel

---

## Summary

Congratulations

You have built a complete production-ready Task Management API

This project includes

```text
User authentication with JWT
Full CRUD operations
File upload for profile pictures
Password reset via email
Task filtering and pagination
Task statistics
Proper error handling
Input validation
MVC architecture
Environment configuration
Security best practices
Testing
```

You are now ready to build real-world backend applications

---