## Table of Contents

* [What is HTTP?](#what-is-http)
* [What is a Client?](#what-is-a-client)
* [What is a Server?](#what-is-a-server)
* [How HTTP Works](#how-http-works)
* [Importing HTTP Module](#importing-http-module)
* [Creating Your First Server](#creating-your-first-server)
* [Understanding Request and Response](#understanding-request-and-response)
* [Creating Routes](#creating-routes)
* [Practice Exercises](#practice-exercises)
* [Interview Questions](#interview-questions)
* [Summary](#summary)

---

## What is HTTP?

HTTP stands for 

```text id="m1i4gr"
HyperText Transfer Protocol
```

HTTP is a communication protocol used between 

```text id="lqnhxg"
- Client
- Server
```

Whenever you open a website, HTTP is used behind the scenes.

Example

```text id="a8s97s"
Browser -> Google Server -> Response
```

---

## What is a Client?

A client is an application that sends requests.

Examples

```text id="qepmxz"
Browser

Mobile App

Postman

Frontend Application
```

Example Flow

```text id="t39lc3"
Client -> Requests Data
```

---

## What is a Server?

A server receives requests and sends responses.

Example Flow

```text id="ocum9u"
Client -> Request

Server -> Response
```

Example

```text id="xjlwmq"
Browser -> Request Homepage

Server -> Returns HTML
```

---

## How HTTP Works

Simple Flow

```text id="zdq8hf"
Client -> Request

Server -> Process Request

Server -> Response

Client -> Displays Result
```

Real Example

```text id="ffupyu"
Open YouTube -> Request Sent

YouTube Server -> Processes Request

YouTube Server -> Returns Data
```

---

## Importing HTTP Module

The HTTP module is built into Node.js.

No installation is required.

Import it using

```javascript id="cbz1g3"
const http = require("http");
```

---

## Creating Your First Server

Create

```text id="1hhrng"
server.js
```

Add

```javascript id="z4tbmf"
const http = require("http");

const server = http.createServer(
  (req, res) => {
    res.end("Hello World");
  }
);

server.listen(3000);
```

Run

```bash id="e0g5aq"
node server.js
```

Output

```text id="lz6xg6"
Server Running On Port 3000
```

Open

```text id="umovqr"
http://localhost:3000
```

Response

```text id="3r5k8x"
Hello World
```

Congratulations.

You have created your first HTTP server.

---

## Understanding createServer()

Example

```javascript id="h9lhrm"
http.createServer((req, res) => {
  res.end("Hello World");
});
```

Flow

```text id="x6wdjx"
Request Received -> Callback Executes -> Response Sent
```

Whenever a request arrives, Node.js executes the callback function.

---

## Understanding Request and Response

The callback receives two objects 

```javascript id="zgrw5z"
(req, res)
```

| Object | Purpose           |
| ------ | ----------------- |
| req    | Incoming Request  |
| res    | Outgoing Response |

---

## Request Object

Useful Properties

```javascript id="i8wjnv"
req.url
```

Returns the requested URL.

Example

```text id="48fjq2"
/about
```

---

```javascript id="zcfbbd"
req.method
```

Returns the HTTP method.

Example

```text id="q25j74"
GET

POST

PUT

DELETE
```

---

## Response Object

Used to send data back to the client.

Example

```javascript id="c0dx0r"
res.end("Welcome");
```

Output

```text id="59iz4n"
Welcome
```

---

## Creating Routes

We can send different responses for different URLs.

Example

```javascript id="d09myk"
const http = require("http");

const server = http.createServer(
  (req, res) => {
    if (req.url === "/") {
      res.end("Home Page");
    } else if (req.url === "/about") {
      res.end("About Page");
    } else {
      res.end("Page Not Found");
    }
  }
);

server.listen(3000);
```

---

## Route Flow

```text id="2dbb6k"
/  -> Home Page

/about -> About Page

Anything Else -> Page Not Found
```

---

## Complete Example

```javascript id="y2v3n3"
const http = require("http");

const server = http.createServer(
  (req, res) => {
    if (req.url === "/") {
      res.end("Welcome To Node.js");
    } else if (req.url === "/contact") {
      res.end("Contact Us");
    } else {
      res.end("404 Not Found");
    }
  }
);

server.listen(3000);
```

Visit 

```text id="1qtjv4"
http://localhost:3000

http://localhost:3000/contact
```

---

## Practice Exercises

### Exercise 1

Create a server that returns 

```text id="r75t5h"
Hello Student
```

for every request.

---

### Exercise 2

Create routes 

```text id="7sv1vx"
/

/about

/contact
```

Return different messages for each route.

---

### Exercise 3

Display 

```javascript id="8a4d9f"
req.url
```

inside the console.

Visit different URLs and observe the output.

---

### Exercise 4

Display 

```javascript id="cjk9tt"
req.method
```

inside the console.

---

## Interview Questions

### What is HTTP?

HTTP stands for HyperText Transfer Protocol and is used for communication between clients and servers.

---

### What is a Client?

A client sends requests to a server.

Examples 

```text id="20wnlz"
Browser

Mobile App

Postman
```

---

### What is a Server?

A server receives requests and sends responses.

---

### What is the purpose of createServer()?

It creates an HTTP server that can handle incoming requests.

---

### What is req?

The request object containing information about the incoming request.

---

### What is res?

The response object used to send data back to the client.

---

### What does res.end() do?

It sends the response and ends the request.

---

## Summary

In this session, you learned 

* What HTTP is
* What a Client is
* What a Server is
* How requests and responses work
* How to create an HTTP server
* How to create basic routes
* How to handle incoming requests

This is the foundation of backend development.

---