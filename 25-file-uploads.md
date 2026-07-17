## Table of Contents

* [What is File Upload](#what-is-file-upload)
* [What is Multer](#what-is-multer)
* [Installing Multer](#installing-multer)
* [Basic File Upload](#basic-file-upload)
* [Single File Upload](#single-file-upload)
* [Multiple File Upload](#multiple-file-upload)
* [File Validation](#file-validation)
* [File Size Limits](#file-size-limits)
* [File Type Validation](#file-type-validation)
* [Custom File Names](#custom-file-names)
* [Serving Static Files](#serving-static-files)
* [Error Handling for Uploads](#error-handling-for-uploads)
* [Complete File Upload API](#complete-file-upload-api)
* [Practice Exercises](#practice-exercises)
* [Interview Questions](#interview-questions)
* [Summary](#summary)

---

## What is File Upload

File upload allows users to send files to your server

Examples of file upload

```text
Profile picture
Document upload
Image gallery
Resume submission
Video upload
```

How file upload works

```text
User selects file on their computer
Browser sends file to server
Server receives file
Server saves file to disk or database
Server sends response back
```

File upload uses multipart/form-data

Different from regular JSON

```text
JSON -> application/json
File -> multipart/form-data
```

---

## What is Multer

Multer is a middleware for handling file uploads

It processes multipart/form-data

Why use Multer

```text
Handles file uploads easily
Saves files to disk or memory
Provides file information
Validates file types and sizes
Renames files automatically
```

Without Multer, handling file uploads is very complex

Multer makes it simple

Think of Multer like a mail sorter

```text
Files come in
Multer sorts them
Multer gives you access to each file
Multer saves them where you want
```

---

## Installing Multer

Create a new project

```bash
mkdir file-upload-api
cd file-upload-api
npm init -y
```

Install packages

```bash
npm install express multer
```

Create uploads folder

```bash
mkdir uploads
```

This folder will store uploaded files

Create server.js file

---

## Basic File Upload

Basic setup with Multer

```javascript
const express = require("express");
const multer = require("multer");
const path = require("path");

const app = express();

// Configure storage
const storage = multer.diskStorage({
  destination: (req, file, cb) => {
    cb(null, "uploads/");
  },
  filename: (req, file, cb) => {
    cb(null, file.originalname);
  }
});

// Create upload middleware
const upload = multer({ storage: storage });

// Upload route
app.post("/upload", upload.single("file"), (req, res) => {
  res.json({
    success: true,
    message: "File uploaded successfully",
    file: req.file
  });
});

// Start server
app.listen(3000, () => {
  console.log("Server running on port 3000");
});
```

Test with curl

```bash
curl -X POST http://localhost:3000/upload \
  -F "file=@/path/to/your/file.jpg"
```

---

## Single File Upload

Upload one file at a time

Use upload.single("fieldName")

```javascript
const express = require("express");
const multer = require("multer");
const path = require("path");

const app = express();

// Configure storage
const storage = multer.diskStorage({
  destination: (req, file, cb) => {
    cb(null, "uploads/");
  },
  filename: (req, file, cb) => {
    // Use original name
    cb(null, file.originalname);
  }
});

const upload = multer({ storage: storage });

// Single file upload
app.post("/upload/profile", upload.single("profileImage"), (req, res) => {
  try {
    if (!req.file) {
      return res.status(400).json({
        success: false,
        message: "No file uploaded"
      });
    }
    
    res.json({
      success: true,
      message: "Profile image uploaded",
      file: {
        name: req.file.filename,
        size: req.file.size,
        type: req.file.mimetype,
        path: req.file.path
      }
    });
  } catch (error) {
    res.status(500).json({
      success: false,
      message: error.message
    });
  }
});

app.listen(3000);
```

Request format

```text
Field name must be "profileImage"
Content-Type must be multipart/form-data
```

---

## Multiple File Upload

Upload multiple files at once

Use upload.array("fieldName", maxCount)

```javascript
// Multiple files with same field name
app.post("/upload/gallery", upload.array("images", 5), (req, res) => {
  try {
    if (!req.files || req.files.length === 0) {
      return res.status(400).json({
        success: false,
        message: "No files uploaded"
      });
    }
    
    const files = req.files.map(file => ({
      name: file.filename,
      size: file.size,
      type: file.mimetype
    }));
    
    res.json({
      success: true,
      message: `${req.files.length} files uploaded`,
      files: files
    });
  } catch (error) {
    res.status(500).json({
      success: false,
      message: error.message
    });
  }
});

// Multiple files with different field names
app.post("/upload/documents", upload.fields([
  { name: "resume", maxCount: 1 },
  { name: "photo", maxCount: 1 },
  { name: "certificates", maxCount: 5 }
]), (req, res) => {
  res.json({
    success: true,
    resume: req.files["resume"],
    photo: req.files["photo"],
    certificates: req.files["certificates"]
  });
});
```

---

## File Validation

Validate files before saving

Check file type, size, etc.

```javascript
const fileFilter = (req, file, cb) => {
  // Allowed file types
  const allowedTypes = ["image/jpeg", "image/png", "image/jpg", "image/gif"];
  
  if (allowedTypes.includes(file.mimetype)) {
    cb(null, true); // Accept file
  } else {
    cb(new Error("Invalid file type. Only images are allowed"), false);
  }
};

const upload = multer({
  storage: storage,
  fileFilter: fileFilter,
  limits: {
    fileSize: 1024 * 1024 * 5 // 5MB limit
  }
});
```

Complete validation example

```javascript
const storage = multer.diskStorage({
  destination: (req, file, cb) => {
    cb(null, "uploads/");
  },
  filename: (req, file, cb) => {
    cb(null, Date.now() + "-" + file.originalname);
  }
});

const fileFilter = (req, file, cb) => {
  const allowedTypes = ["image/jpeg", "image/png", "image/jpg"];
  
  if (allowedTypes.includes(file.mimetype)) {
    cb(null, true);
  } else {
    cb(new Error("Only JPEG and PNG images are allowed"), false);
  }
};

const upload = multer({
  storage: storage,
  fileFilter: fileFilter,
  limits: {
    fileSize: 1024 * 1024 * 2 // 2MB
  }
});

app.post("/upload/avatar", upload.single("avatar"), (req, res) => {
  res.json({
    success: true,
    message: "Avatar uploaded",
    file: req.file
  });
});
```

---

## File Size Limits

Set limits on file sizes

```javascript
const upload = multer({
  storage: storage,
  limits: {
    fileSize: 1024 * 1024 * 5, // 5MB
    files: 5 // Max number of files
  }
});

// Different limits for different routes
const profileUpload = multer({
  storage: storage,
  limits: { fileSize: 1024 * 1024 * 1 } // 1MB for profile
});

const videoUpload = multer({
  storage: storage,
  limits: { fileSize: 1024 * 1024 * 100 } // 100MB for videos
});

app.post("/upload/profile", profileUpload.single("photo"), handler);
app.post("/upload/video", videoUpload.single("video"), handler);
```

---

## File Type Validation

Check file types by extension and mime type

```javascript
const allowedFileTypes = {
  image: ["image/jpeg", "image/png", "image/gif"],
  document: ["application/pdf", "application/msword"],
  video: ["video/mp4", "video/mpeg"]
};

const fileFilter = (req, file, cb) => {
  // Check by mimetype
  if (file.mimetype.startsWith("image/")) {
    cb(null, true);
  } else {
    cb(new Error("Only images are allowed"), false);
  }
};

// Check by extension
const fileFilterByExtension = (req, file, cb) => {
  const ext = path.extname(file.originalname).toLowerCase();
  
  if (ext === ".jpg" || ext === ".jpeg" || ext === ".png") {
    cb(null, true);
  } else {
    cb(new Error("Only .jpg, .jpeg, .png files are allowed"), false);
  }
};
```

---

## Custom File Names

Generate custom names for uploaded files

```javascript
const storage = multer.diskStorage({
  destination: (req, file, cb) => {
    cb(null, "uploads/");
  },
  filename: (req, file, cb) => {
    // Method 1: Use timestamp
    const uniqueSuffix = Date.now() + "-" + Math.round(Math.random() * 1E9);
    cb(null, uniqueSuffix + path.extname(file.originalname));
    
    // Method 2: Use user id
    // const userId = req.user.id;
    // cb(null, userId + "-" + file.originalname);
    
    // Method 3: Use original name with timestamp
    // const name = file.originalname.split(".")[0];
    // const ext = path.extname(file.originalname);
    // cb(null, name + "-" + Date.now() + ext);
  }
});

// Create folders based on file type
const dynamicStorage = multer.diskStorage({
  destination: (req, file, cb) => {
    let folder = "uploads/";
    
    if (file.mimetype.startsWith("image/")) {
      folder += "images/";
    } else if (file.mimetype.startsWith("video/")) {
      folder += "videos/";
    } else {
      folder += "documents/";
    }
    
    cb(null, folder);
  },
  filename: (req, file, cb) => {
    cb(null, Date.now() + "-" + file.originalname);
  }
});
```

---

## Serving Static Files

Make uploaded files accessible to users

```javascript
const express = require("express");
const path = require("path");

const app = express();

// Serve static files from uploads folder
app.use("/uploads", express.static(path.join(__dirname, "uploads")));

// Now users can access files via URL
// http://localhost:3000/uploads/image.jpg
```

Example response with file URL

```javascript
app.post("/upload", upload.single("file"), (req, res) => {
  const fileUrl = `${req.protocol}://${req.get("host")}/uploads/${req.file.filename}`;
  
  res.json({
    success: true,
    url: fileUrl,
    file: req.file
  });
});
```

---

## Error Handling for Uploads

Handle Multer errors properly

```javascript
const upload = multer({
  storage: storage,
  fileFilter: fileFilter,
  limits: { fileSize: 1024 * 1024 * 5 }
});

// Error handling middleware for Multer
app.post("/upload", (req, res) => {
  upload.single("file")(req, res, (err) => {
    if (err instanceof multer.MulterError) {
      // Multer specific errors
      if (err.code === "FILE_TOO_LARGE") {
        return res.status(400).json({
          success: false,
          message: "File too large. Max size is 5MB"
        });
      }
      if (err.code === "LIMIT_FILE_COUNT") {
        return res.status(400).json({
          success: false,
          message: "Too many files"
        });
      }
    }
    
    if (err) {
      return res.status(400).json({
        success: false,
        message: err.message
      });
    }
    
    if (!req.file) {
      return res.status(400).json({
        success: false,
        message: "No file uploaded"
      });
    }
    
    res.json({
      success: true,
      file: req.file
    });
  });
});
```

---

## Complete File Upload API

```javascript
const express = require("express");
const multer = require("multer");
const path = require("path");
const fs = require("fs");

const app = express();

// Create uploads folder if it doesn't exist
const uploadDir = "uploads";
if (!fs.existsSync(uploadDir)) {
  fs.mkdirSync(uploadDir);
}

// Configure storage
const storage = multer.diskStorage({
  destination: (req, file, cb) => {
    let folder = "uploads/";
    
    if (file.mimetype.startsWith("image/")) {
      folder += "images";
    } else if (file.mimetype.startsWith("video/")) {
      folder += "videos";
    } else {
      folder += "documents";
    }
    
    // Create folder if it doesn't exist
    if (!fs.existsSync(folder)) {
      fs.mkdirSync(folder, { recursive: true });
    }
    
    cb(null, folder);
  },
  filename: (req, file, cb) => {
    const uniqueName = Date.now() + "-" + Math.round(Math.random() * 1E9);
    const ext = path.extname(file.originalname);
    cb(null, uniqueName + ext);
  }
});

// File filter
const fileFilter = (req, file, cb) => {
  const allowedImages = ["image/jpeg", "image/png", "image/jpg", "image/gif"];
  const allowedDocs = ["application/pdf", "application/msword"];
  const allowedVideos = ["video/mp4", "video/mpeg"];
  
  const allowed = [...allowedImages, ...allowedDocs, ...allowedVideos];
  
  if (allowed.includes(file.mimetype)) {
    cb(null, true);
  } else {
    cb(new Error("File type not allowed"), false);
  }
};

// Create multer instance
const upload = multer({
  storage: storage,
  fileFilter: fileFilter,
  limits: {
    fileSize: 1024 * 1024 * 10, // 10MB
    files: 5
  }
});

// Serve static files
app.use("/uploads", express.static("uploads"));

// Routes

// Single file upload
app.post("/upload/single", upload.single("file"), (req, res) => {
  try {
    if (!req.file) {
      return res.status(400).json({
        success: false,
        message: "No file uploaded"
      });
    }
    
    const fileUrl = `${req.protocol}://${req.get("host")}/${req.file.path}`;
    
    res.json({
      success: true,
      message: "File uploaded successfully",
      file: {
        name: req.file.filename,
        size: req.file.size,
        type: req.file.mimetype,
        path: req.file.path,
        url: fileUrl
      }
    });
  } catch (error) {
    res.status(500).json({
      success: false,
      message: error.message
    });
  }
});

// Multiple files upload
app.post("/upload/multiple", upload.array("files", 5), (req, res) => {
  try {
    if (!req.files || req.files.length === 0) {
      return res.status(400).json({
        success: false,
        message: "No files uploaded"
      });
    }
    
    const files = req.files.map(file => ({
      name: file.filename,
      size: file.size,
      type: file.mimetype,
      url: `${req.protocol}://${req.get("host")}/${file.path}`
    }));
    
    res.json({
      success: true,
      message: `${req.files.length} files uploaded`,
      count: req.files.length,
      files: files
    });
  } catch (error) {
    res.status(500).json({
      success: false,
      message: error.message
    });
  }
});

// Different fields upload
app.post("/upload/fields", upload.fields([
  { name: "avatar", maxCount: 1 },
  { name: "documents", maxCount: 3 }
]), (req, res) => {
  res.json({
    success: true,
    avatar: req.files["avatar"] ? req.files["avatar"][0] : null,
    documents: req.files["documents"] || []
  });
});

// Get all uploaded files
app.get("/files", (req, res) => {
  const getAllFiles = (dir) => {
    const files = [];
    const items = fs.readdirSync(dir, { withFileTypes: true });
    
    for (const item of items) {
      const fullPath = path.join(dir, item.name);
      if (item.isDirectory()) {
        files.push(...getAllFiles(fullPath));
      } else {
        files.push({
          name: item.name,
          path: fullPath,
          size: fs.statSync(fullPath).size
        });
      }
    }
    
    return files;
  };
  
  const allFiles = getAllFiles("uploads");
  
  res.json({
    success: true,
    count: allFiles.length,
    files: allFiles
  });
});

// Delete file
app.delete("/files/:filename", (req, res) => {
  const filename = req.params.filename;
  
  // Search for file in all subfolders
  const findAndDelete = (dir) => {
    const items = fs.readdirSync(dir, { withFileTypes: true });
    
    for (const item of items) {
      const fullPath = path.join(dir, item.name);
      if (item.isDirectory()) {
        findAndDelete(fullPath);
      } else if (item.name === filename) {
        fs.unlinkSync(fullPath);
        return true;
      }
    }
    return false;
  };
  
  const deleted = findAndDelete("uploads");
  
  if (deleted) {
    res.json({
      success: true,
      message: "File deleted successfully"
    });
  } else {
    res.status(404).json({
      success: false,
      message: "File not found"
    });
  }
});

// Error handling for Multer
app.use((err, req, res, next) => {
  if (err instanceof multer.MulterError) {
    if (err.code === "FILE_TOO_LARGE") {
      return res.status(400).json({
        success: false,
        message: "File too large. Max size is 10MB"
      });
    }
    if (err.code === "LIMIT_FILE_COUNT") {
      return res.status(400).json({
        success: false,
        message: "Too many files. Max is 5"
      });
    }
    if (err.code === "LIMIT_UNEXPECTED_FILE") {
      return res.status(400).json({
        success: false,
        message: "Unexpected field name"
      });
    }
  }
  
  if (err) {
    return res.status(400).json({
      success: false,
      message: err.message
    });
  }
  
  next();
});

const PORT = 3000;
app.listen(PORT, () => {
  console.log(`Server running on port ${PORT}`);
  console.log(`Upload endpoint: http://localhost:${PORT}/upload/single`);
  console.log(`View files: http://localhost:${PORT}/files`);
});
```

---

## Testing File Upload

Using curl

Single file upload

```bash
curl -X POST http://localhost:3000/upload/single \
  -F "file=@/path/to/image.jpg"
```

Multiple files

```bash
curl -X POST http://localhost:3000/upload/multiple \
  -F "files=@/path/to/file1.jpg" \
  -F "files=@/path/to/file2.pdf"
```

View all files

```bash
curl http://localhost:3000/files
```

Delete a file

```bash
curl -X DELETE http://localhost:3000/files/1703123456789-123456789.jpg
```

Using Postman

```text
Method: POST
URL: http://localhost:3000/upload/single
Body -> form-data
Key: file (type: File)
Value: Select a file from your computer
```

---

## Practice Exercises

### Exercise 1

Create a profile picture upload

Only allow images

Max size 1MB

Save with user id as filename

### Exercise 2

Create a document upload system

Allow PDF and DOC files

Max size 5MB

Store in documents folder

### Exercise 3

Add file validation to check for viruses

Check file signature (magic numbers)

### Exercise 4

Create a gallery upload

Allow up to 10 images

Create thumbnails for each image

### Exercise 5

Implement file sharing system

Users can upload files

Get unique download link

Link expires after 24 hours

---

## Interview Questions

### What is Multer

Multer is middleware for handling multipart/form-data file uploads in Express

### What is the difference between single and array methods

single handles one file with specific field name
array handles multiple files with same field name

### What is diskStorage

diskStorage is a Multer storage engine that saves files to the disk

### How do you validate file types

Using fileFilter function that checks mimetype or file extension

### How do you limit file size

Using limits.fileSize option in Multer configuration

### Where should uploaded files be stored

Uploads folder on disk or cloud storage like S3

### How do you serve uploaded files

Using express.static middleware to serve the uploads folder

### What is the difference between req.file and req.files

req.file contains single uploaded file
req.files contains array or object of multiple files

---

## Summary

In this session, you learned

* What file upload is
* What Multer is and why to use it
* How to configure Multer storage
* Single file upload
* Multiple file upload
* File validation (type and size)
* Custom file naming
* Serving static files
* Error handling for uploads
* Complete file upload API

You can now handle file uploads in your applications

---