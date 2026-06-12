## Table of Contents

* [Project Overview](#project-overview)
* [Project Structure](#project-structure)
* [Creating the Server](#creating-the-server)
* [Adding Routes](#adding-routes)
* [Returning JSON Data](#returning-json-data)
* [Using Status Codes](#using-status-codes)
* [Handling 404 Routes](#handling-404-routes)
* [Complete Example](#complete-example)
* [Practice Exercises](#practice-exercises)
* [Interview Questions](#interview-questions)
* [Summary](#summary)

---

## Project Overview

In the previous session, we created a simple server.

In this session, we will create a server with multiple routes.

Routes

```text
/

about

/contact

/products
```

Each route will return a different response.

---

## Project Structure

```text
first-server-project/

├── server.js
└── package.json
```

---

## Creating the Server

Create

```text
server.js
```

Add

```javascript
const http = require("http");

const server = http.createServer((req, res) => {
  res.end("Server Running");
});

server.listen(3000);
```

Run

```bash
node server.js
```

Open

```text
http://localhost:3000
```

---

## Adding Routes

We can check the requested URL using

```javascript
req.url
```

Example

```javascript
const http = require("http");

const server = http.createServer((req, res) => {
  if (req.url === "/") {
    res.end("Home Page");
  } else if (req.url === "/about") {
    res.end("About Page");
  } else {
    res.end("Page Not Found");
  }
});

server.listen(3000);
```

---

## Route Flow

```text
/ -> Home Page

/about -> About Page

/contact -> Contact Page
```

---

## Returning JSON Data

Backend applications commonly return JSON.

Example Data

```javascript
const products = [
  {
    id: 1,
    name: "Laptop"
  },
  {
    id: 2,
    name: "Mobile"
  }
];
```

Response

```javascript
res.setHeader(
  "Content-Type",
  "application/json"
);

res.end(JSON.stringify(products));
```

Output

```json
[
  {
    "id": 1,
    "name": "Laptop"
  },
  {
    "id": 2,
    "name": "Mobile"
  }
]
```

---

## Using Status Codes

Status codes tell the client whether a request was successful.

Common Status Codes

| Code | Meaning      |
| ---- | ------------ |
| 200  | Success      |
| 404  | Not Found    |
| 500  | Server Error |

Example

```javascript
res.statusCode = 200;

res.end("Success");
```

---

## Handling 404 Routes

When a route does not exist

```javascript
res.statusCode = 404;

res.end("Page Not Found");
```

Example

```javascript
if (req.url === "/") {
  res.end("Home Page");
} else {
  res.statusCode = 404;

  res.end("Page Not Found");
}
```

---

## Complete Example

```javascript
const http = require("http");

const products = [
  {
    id: 1,
    name: "Laptop"
  },
  {
    id: 2,
    name: "Mobile"
  }
];

const server = http.createServer((req, res) => {
  if (req.url === "/") {
    res.statusCode = 200;

    res.end("Welcome To My First Server");
  } else if (req.url === "/about") {
    res.statusCode = 200;

    res.end("About Page");
  } else if (req.url === "/contact") {
    res.statusCode = 200;

    res.end("Contact Page");
  } else if (req.url === "/products") {
    res.statusCode = 200;

    res.setHeader(
      "Content-Type",
      "application/json"
    );

    res.end(JSON.stringify(products));
  } else {
    res.statusCode = 404;

    res.end("Page Not Found");
  }
});

server.listen(3000, () => {
  console.log("Server Running On Port 3000");
});
```

---

## Testing Routes

Visit

```text
http://localhost:3000/
```

Output

```text
Welcome To My First Server
```

---

Visit

```text
http://localhost:3000/about
```

Output

```text
About Page
```

---

Visit

```text
http://localhost:3000/contact
```

Output

```text
Contact Page
```

---

Visit

```text
http://localhost:3000/products
```

Output

```json
[
  {
    "id": 1,
    "name": "Laptop"
  },
  {
    "id": 2,
    "name": "Mobile"
  }
]
```

---

Visit

```text
http://localhost:3000/random
```

Output

```text
Page Not Found
```

---

## Practice Exercises

### Exercise 1

Create routes 

```text
/

/about

/services
```

Return different responses for each route.

---

### Exercise 2

Create 

```text
/students
```

Return the following JSON

```json
[
  {
    "id": 1,
    "name": "John"
  },
  {
    "id": 2,
    "name": "Alex"
  }
]
```

---

### Exercise 3

Handle invalid routes using 

```text
404
```

status code.

---

### Exercise 4

Print the requested URL in the terminal.

Example

```javascript
console.log(req.url);
```

---

## Interview Questions

### What is a route?

A route is a URL path used to access a specific resource.

Example

```text
/

/about

/products
```

---

### What is req.url?

It returns the requested URL path.

---

### Why do we use JSON?

JSON is the most common format used to exchange data between clients and servers.

---

### What is a status code?

A status code tells the client whether a request was successful or failed.

---

### What does status code 404 mean?

The requested resource was not found.

---

## Summary

In this session, you learned

* How to create a multi-route server
* How to handle URLs
* How to send JSON responses
* How to use status codes
* How to handle invalid routes

You have now built your first small backend application using only Node.js.

---