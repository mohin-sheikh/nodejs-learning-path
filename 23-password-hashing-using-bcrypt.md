## Table of Contents

* [Why Password Hashing](#why-password-hashing)
* [What is bcrypt](#what-is-bcrypt)
* [Hashing vs Encryption](#hashing-vs-encryption)
* [How bcrypt Works](#how-bcrypt-works)
* [Installing bcrypt](#installing-bcrypt)
* [Hashing a Password](#hashing-a-password)
* [Comparing a Password](#comparing-a-password)
* [Salt Rounds](#salt-rounds)
* [Complete Password Management Example](#complete-password-management-example)
* [Practice Exercises](#practice-exercises)
* [Interview Questions](#interview-questions)
* [Summary](#summary)

---

## Why Password Hashing

Storing passwords in plain text is dangerous

Never do this

```javascript
const user = {
  name: "John",
  password: "123456"  // DANGER! Never do this
};
```

What if someone gets access to your database

```text
Hacker steals database
Sees all passwords in plain text
Can login as any user
Users use same password on other sites
Huge security problem
```

Solution - Hash the password

```text
User enters password "123456"
Server hashes it to "fj3k9d8f3j9d8f3j9d8f3j9d8f"
Store hash in database
Even if hacker steals database
They cannot get original password
```

---

## What is bcrypt

bcrypt is a password hashing library

It is designed specifically for passwords

Features of bcrypt

```text
Slow by design (makes brute force hard)
Adds salt automatically (prevents rainbow tables)
Produces same length output always
Widely used and trusted
```

bcrypt vs simple hash

Simple hash like MD5 or SHA256

```text
Fast to compute
Same password always gives same hash
Hackers can pre-compute hashes (rainbow tables)
```

bcrypt

```text
Slow to compute (takes time)
Adds random salt
Same password gives different hash each time
Much more secure
```

---

## Hashing vs Encryption

People often confuse hashing and encryption

They are different

Encryption

```text
Two-way process
Can encrypt and decrypt
Needs a key
Example: HTTPS, file encryption
```

Hashing

```text
One-way process
Cannot reverse
No key needed
Example: Passwords, digital signatures
```

Comparison table

| Feature | Hashing | Encryption |
| ------- | ------- | ---------- |
| Reversible | No | Yes |
| Needs key | No | Yes |
| Same input same output | Yes (for same hash) | No (with different key) |
| Use case | Passwords | Secure communication |

Think of hashing like making a smoothie

```text
Put in fruits -> Get smoothie
Cannot get fruits back from smoothie
Same fruits always make same smoothie
```

Think of encryption like a lockbox

```text
Put item in box -> Lock with key
Key opens box -> Get item back
```

---

## How bcrypt Works

bcrypt has three steps

Step 1 - Generate a salt

```text
Salt is random data
Makes each hash unique
Even for same password
```

Step 2 - Combine password and salt

```text
password + salt = combined
```

Step 3 - Hash the combined string

```text
bcrypt(combined) = final hash
```

The final hash contains

```text
Algorithm version
Salt
Hash value
All in one string
```

Example bcrypt hash

```text
$2b$10$abcdefghij1234567890u7xYz8ABCDEFGHIJKLMNOPQRSTUVWXYZ
```

Parts of the hash

```text
$2b$ -> Algorithm version
10 -> Salt rounds (2^10 iterations)
abcdefghij1234567890 -> Salt (22 characters)
u7xYz8ABCDEFGHIJKLMNOPQRSTUVWXYZ -> Hash (31 characters)
```

---

## Installing bcrypt

bcrypt has two versions

```text
bcrypt -> Written in C++ (faster but needs compilation)
bcryptjs -> Pure JavaScript (easier to install)
```

For beginners, bcryptjs is better

No compilation needed

Works on all systems

Install bcryptjs

```bash
npm install bcryptjs
```

For production apps, use bcrypt

```bash
npm install bcrypt
```

We will use bcryptjs for this session

---

## Hashing a Password

Basic hashing

```javascript
const bcrypt = require("bcryptjs");

const password = "mySecretPassword123";

bcrypt.genSalt(10, (err, salt) => {
  bcrypt.hash(password, salt, (err, hash) => {
    console.log("Hash:", hash);
  });
});
```

Using promises (easier)

```javascript
const bcrypt = require("bcryptjs");

async function hashPassword() {
  const password = "mySecretPassword123";
  
  const salt = await bcrypt.genSalt(10);
  const hash = await bcrypt.hash(password, salt);
  
  console.log("Password:", password);
  console.log("Hash:", hash);
}

hashPassword();
```

Output example

```text
Password: mySecretPassword123
Hash: $2a$10$N9qo8uLOickgx2ZMRZoMy.Mr5v7vXvK9vXvK9vXvK9vXvK9vXvK9
```

Same password, different hash each time

```javascript
// First run
const hash1 = await bcrypt.hash("password123", await bcrypt.genSalt(10));

// Second run
const hash2 = await bcrypt.hash("password123", await bcrypt.genSalt(10));

console.log(hash1 === hash2); // false (different salts)
```

---

## Comparing a Password

When user logs in

```text
User enters password
Get stored hash from database
Compare entered password with stored hash
bcrypt handles the salt automatically
```

Compare function

```javascript
const bcrypt = require("bcryptjs");

async function comparePasswords() {
  const enteredPassword = "mySecretPassword123";
  const storedHash = "$2a$10$N9qo8uLOickgx2ZMRZoMy.Mr5v7vXvK9vXvK9vXvK9vXvK9vXvK9";
  
  const isMatch = await bcrypt.compare(enteredPassword, storedHash);
  
  if (isMatch) {
    console.log("Password is correct");
  } else {
    console.log("Password is incorrect");
  }
}

comparePasswords();
```

How bcrypt.compare works

```text
Extracts salt from stored hash
Hashes entered password with same salt
Compares results
Returns true or false
```

---

## Salt Rounds

Salt rounds determine how slow bcrypt is

Higher salt rounds = slower but more secure

Formula

```text
Time = 2^saltRounds milliseconds
```

Examples

| Salt Rounds | Approximate Time |
| ----------- | ----------------- |
| 8 | 0.04 seconds |
| 10 | 0.08 seconds |
| 12 | 0.32 seconds |
| 14 | 1.28 seconds |
| 16 | 5.12 seconds |

Recommended salt rounds

```text
Development -> 10
Production (low security) -> 10
Production (normal) -> 12
Production (high security) -> 14
```

Too high salt rounds cause slow response times

Example with different salt rounds

```javascript
const bcrypt = require("bcryptjs");

async function testSaltRounds() {
  const password = "test123";
  
  // Salt rounds 8
  console.time("Salt 8");
  const hash8 = await bcrypt.hash(password, 8);
  console.timeEnd("Salt 8");
  
  // Salt rounds 10
  console.time("Salt 10");
  const hash10 = await bcrypt.hash(password, 10);
  console.timeEnd("Salt 10");
  
  // Salt rounds 12
  console.time("Salt 12");
  const hash12 = await bcrypt.hash(password, 12);
  console.timeEnd("Salt 12");
}

testSaltRounds();
```

Output example

```text
Salt 8: 42ms
Salt 10: 85ms
Salt 12: 340ms
```

---

## Complete Password Management Example

Create a file named passwordManager.js

```javascript
const bcrypt = require("bcryptjs");

// Simulate database
const users = [];

// Function to register a new user
async function registerUser(name, email, password) {
  try {
    // Hash the password
    const saltRounds = 10;
    const hashedPassword = await bcrypt.hash(password, saltRounds);
    
    // Store user in database
    const newUser = {
      id: users.length + 1,
      name: name,
      email: email,
      password: hashedPassword,
      createdAt: new Date()
    };
    
    users.push(newUser);
    
    console.log("User registered successfully");
    console.log("Stored hash:", hashedPassword);
    
    return newUser;
    
  } catch (error) {
    console.error("Registration failed:", error);
  }
}

// Function to login a user
async function loginUser(email, password) {
  try {
    // Find user in database
    const user = users.find(u => u.email === email);
    
    if (!user) {
      console.log("User not found");
      return false;
    }
    
    // Compare password
    const isMatch = await bcrypt.compare(password, user.password);
    
    if (isMatch) {
      console.log("Login successful!");
      console.log(`Welcome back, ${user.name}`);
      return true;
    } else {
      console.log("Incorrect password");
      return false;
    }
    
  } catch (error) {
    console.error("Login failed:", error);
    return false;
  }
}

// Function to change password
async function changePassword(email, oldPassword, newPassword) {
  try {
    const user = users.find(u => u.email === email);
    
    if (!user) {
      console.log("User not found");
      return false;
    }
    
    // Verify old password
    const isMatch = await bcrypt.compare(oldPassword, user.password);
    
    if (!isMatch) {
      console.log("Old password is incorrect");
      return false;
    }
    
    // Hash new password
    const newHashedPassword = await bcrypt.hash(newPassword, 10);
    
    // Update password
    user.password = newHashedPassword;
    user.updatedAt = new Date();
    
    console.log("Password changed successfully");
    return true;
    
  } catch (error) {
    console.error("Password change failed:", error);
    return false;
  }
}

// Function to verify password strength
function isStrongPassword(password) {
  const minLength = password.length >= 8;
  const hasUpperCase = /[A-Z]/.test(password);
  const hasLowerCase = /[a-z]/.test(password);
  const hasNumbers = /[0-9]/.test(password);
  const hasSpecialChar = /[!@#$%^&*(),.?":{}|<>]/.test(password);
  
  return minLength && hasUpperCase && hasLowerCase && hasNumbers && hasSpecialChar;
}

// Demo
async function runDemo() {
  console.log("=== Password Hashing Demo ===\n");
  
  // Register a user
  console.log("1. Registering user...");
  await registerUser("John Doe", "john@example.com", "MySecurePass123!");
  
  console.log("\n2. Attempting login with correct password...");
  await loginUser("john@example.com", "MySecurePass123!");
  
  console.log("\n3. Attempting login with wrong password...");
  await loginUser("john@example.com", "WrongPassword");
  
  console.log("\n4. Changing password...");
  await changePassword("john@example.com", "MySecurePass123!", "NewPass456!");
  
  console.log("\n5. Login with new password...");
  await loginUser("john@example.com", "NewPass456!");
  
  console.log("\n6. Testing password strength...");
  const weakPasswords = ["123", "password", "12345678"];
  const strongPasswords = ["StrongP@ss123!", "Secure#Pass456"];
  
  console.log("Weak passwords:");
  weakPasswords.forEach(p => {
    console.log(`  "${p}" -> ${isStrongPassword(p) ? "Strong" : "Weak"}`);
  });
  
  console.log("\nStrong passwords:");
  strongPasswords.forEach(p => {
    console.log(`  "${p}" -> ${isStrongPassword(p) ? "Strong" : "Weak"}`);
  });
}

runDemo();
```

Run the demo

```bash
node passwordManager.js
```

---

## bcrypt in Express (Review from Session 22)

In our User model, we used bcrypt

```javascript
// models/User.js
const bcrypt = require("bcryptjs");

userSchema.pre("save", async function(next) {
  if (!this.isModified("password")) {
    return next();
  }
  
  const salt = await bcrypt.genSalt(10);
  this.password = await bcrypt.hash(this.password, salt);
  next();
});

userSchema.methods.comparePassword = async function(enteredPassword) {
  return await bcrypt.compare(enteredPassword, this.password);
};
```

In authController

```javascript
// controllers/authController.js
const login = async (req, res) => {
  const user = await User.findOne({ email }).select("+password");
  
  // bcrypt compares automatically
  const isPasswordMatch = await user.comparePassword(password);
  
  if (!isPasswordMatch) {
    return res.status(401).json({ message: "Invalid credentials" });
  }
};
```

---

## Security Best Practices

Always hash passwords

```text
Never store plain text passwords
Always use bcrypt or similar
Salt rounds minimum 10
```

Use strong passwords

```text
Minimum 8 characters
Mix of uppercase and lowercase
Include numbers
Include special characters
```

Additional security

```text
Rate limit login attempts
Use HTTPS in production
Implement account lockout after failed attempts
Use two-factor authentication for sensitive apps
```

What never to do

```text
Don't use MD5 or SHA1 for passwords (too fast)
Don't create your own hashing algorithm
Don't store passwords in logs
Don't send passwords in URLs
Don't log passwords in console
```

---

## Practice Exercises

### Exercise 1

Create a function that hashes a password and prints

```text
Original password
Salt rounds used
Generated hash
Time taken to hash
```

### Exercise 2

Create a function that tests different salt rounds

Test with 8, 10, 12, 14

Measure time for each

### Exercise 3

Create a password strength meter

Return score from 0 to 100

Based on length, character types, etc.

### Exercise 4

Add password confirmation to registration

Check if password and confirmPassword match

### Exercise 5

Implement forgot password feature

Generate reset token

Hash and save new password

---

## Interview Questions

### Why do we hash passwords

To protect passwords if database is compromised

### What is the difference between hashing and encryption

Hashing is one-way, cannot be reversed
Encryption is two-way, can be decrypted with key

### What is a salt in bcrypt

Random data added to password before hashing to make each hash unique

### What are salt rounds in bcrypt

Number of iterations used to hash the password, higher = more secure but slower

### Why is bcrypt better than MD5 or SHA

bcrypt is slow by design and includes salt automatically
MD5 and SHA are too fast for passwords

### Can two identical passwords have different bcrypt hashes

Yes
Because of random salt added to each hash

### How does bcrypt compare work

It extracts salt from stored hash, hashes input password with same salt, then compares

### What happens if you lose the salt

You cannot verify the password
This is why salt is stored with the hash

---

## Summary

In this session, you learned

* Why password hashing is important
* What bcrypt is and how it works
* Difference between hashing and encryption
* How to hash passwords with bcrypt
* How to compare passwords with bcrypt
* What salt rounds are and how to choose them
* How to implement password hashing in Express
* Password security best practices

Never store plain text passwords

Always use bcrypt

---