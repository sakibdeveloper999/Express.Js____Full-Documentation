
---

# 🚀 **Setting Up the Express.js Environment (English + বাংলা)**

---

## 🧩 **1. What is Express.js?**

**English:**
Express.js is a lightweight and fast **web application framework** for Node.js that simplifies building web servers and APIs.

**Bangla:**
Express.js হলো Node.js এর জন্য একটি **লাইটওয়েট ও দ্রুত ওয়েব অ্যাপ্লিকেশন ফ্রেমওয়ার্ক**, যা সার্ভার ও API তৈরি করা অনেক সহজ করে তোলে।

---

## ⚙️ **2. Prerequisites (আগে যা দরকার)**

**English:**
You need Node.js and npm installed on your computer.

**Bangla:**
তোমার কম্পিউটারে Node.js এবং npm ইনস্টল থাকতে হবে।

**Check installation:**

```bash
node -v
npm -v
```

If not installed, download from 👉 [https://nodejs.org](https://nodejs.org)

---

## 📁 **3. Create a New Project Folder (নতুন প্রজেক্ট ফোল্ডার তৈরি)**

```bash
mkdir expressjs-app
cd expressjs-app
```

**English:** Creates and opens a new folder for your Express project.
**Bangla:** এটি একটি নতুন ফোল্ডার তৈরি করে যেখানে তোমার Express প্রজেক্ট থাকবে।

---

## 📦 **4. Initialize Node.js Project (Node.js প্রজেক্ট ইনিশিয়ালাইজ করা)**

```bash
npm init -y
```

**English:** This creates a `package.json` file for your project configuration.
**Bangla:** এটি তোমার প্রজেক্টের জন্য একটি `package.json` ফাইল তৈরি করবে যা প্রজেক্টের সব কনফিগারেশন রাখবে।

Example:

```json
{
  "name": "expressjs-app",
  "version": "1.0.0",
  "scripts": {
    "start": "node index.js"
  }
}
```

---

## 🌐 **5. Install Express.js (Express.js ইনস্টল করা)**

```bash
npm install express
```

**English:** Installs Express into your project.
**Bangla:** এটি তোমার প্রজেক্টে Express ইনস্টল করবে।

---

## 🧠 **6. Create Your First Express App (প্রথম Express অ্যাপ তৈরি)**

Create a new file:

```bash
touch index.js
```

Add this code:

```js
const express = require('express');
const app = express();

app.get('/', (req, res) => {
  res.send('Hello, Express.js!');
});

const PORT = 3000;
app.listen(PORT, () => {
  console.log(`Server running on http://localhost:${PORT}`);
});
```

**English:** This code sets up a simple server and sends “Hello, Express.js!” when you open it in your browser.
**Bangla:** এই কোডটি একটি সহজ সার্ভার তৈরি করবে এবং ব্রাউজারে “Hello, Express.js!” দেখাবে।

---

## ▶️ **7. Run the Server (সার্ভার চালানো)**

```bash
node index.js
```

Then open:
👉 [http://localhost:3000](http://localhost:3000)

**Output:**

```
Hello, Express.js!
```

**English:** Your Express environment is ready!
**Bangla:** তোমার Express এনভায়রনমেন্ট এখন প্রস্তুত!

---

## 🔁 **8. Auto Restart using Nodemon (Nodemon দিয়ে অটো রিস্টার্ট)**

Install:

```bash
npm install -g nodemon
```

or (as dev dependency)

```bash
npm install --save-dev nodemon
```

Update `package.json`:

```json
"scripts": {
  "start": "node index.js",
  "dev": "nodemon index.js"
}
```

Run:

```bash
npm run dev
```

**English:** Server auto-restarts on file changes.
**Bangla:** ফাইল পরিবর্তন করলে সার্ভার নিজে থেকে রিস্টার্ট হবে।

---

## 🧩 **9. Recommended Folder Structure (রেকমেন্ডেড ফোল্ডার স্ট্রাকচার)**

```
expressjs-app/
├── node_modules/
├── routes/
│   └── userRoutes.js
├── public/
│   └── styles.css
├── views/
│   └── index.ejs
├── index.js
├── package.json
└── package-lock.json
```

**English:** Helps to organize files better in large apps.
**Bangla:** বড় অ্যাপে ফাইলগুলো সংগঠিতভাবে রাখার জন্য এই স্ট্রাকচার ব্যবহার করা হয়।

---

## ⚡ **10. Add Middleware Example (Middleware উদাহরণ)**

```js
app.use(express.json());

app.use((req, res, next) => {
  console.log(`${req.method} ${req.url}`);
  next();
});
```

**English:** Middleware runs between request and response.
**Bangla:** Middleware হলো এমন কোড যা রিকুয়েস্ট ও রেসপন্সের মাঝে কাজ করে।

---

## 🔄 **11. Add More Routes (আরও রাউট যোগ করা)**

```js
app.get('/about', (req, res) => {
  res.send('About Page');
});

app.get('/contact', (req, res) => {
  res.send('Contact Page');
});
```

**English:** You can now open multiple routes in browser.
**Bangla:** এখন তুমি ব্রাউজারে একাধিক পেইজ (রাউট) খুলতে পারবে।

---

## ✅ **12. Common Commands Summary (গুরুত্বপূর্ণ কমান্ড সারাংশ)**

| Command               | Description (English) | বর্ণনা (Bangla)          |
| --------------------- | --------------------- | ------------------------ |
| `npm init -y`         | Create package.json   | প্রজেক্ট সেটআপ ফাইল তৈরি |
| `npm install express` | Install Express       | Express ইনস্টল           |
| `node index.js`       | Run server            | সার্ভার চালানো           |
| `npm run dev`         | Run with nodemon      | Nodemon দিয়ে চালানো     |
| `ctrl + c`            | Stop server           | সার্ভার বন্ধ করা         |

---

## 🧾 **13. Troubleshooting Tips (সমস্যা সমাধান)**

| Problem                      | English Solution              | Bangla Solution                 |
| ---------------------------- | ----------------------------- | ------------------------------- |
| Cannot find module 'express' | Run `npm install express`     | আবার Express ইনস্টল করো         |
| Port already in use          | Change port number            | অন্য পোর্ট নম্বর ব্যবহার করো    |
| JSON not parsing             | Add `app.use(express.json())` | JSON পার্সার middleware যোগ করো |

---

## 🧠 **14. What You Learned (তুমি কী শিখলে)**

✅ Install and configure Express.js
✅ Build your first server
✅ Add routes and middleware
✅ Use Nodemon for auto-reload

**Bangla:**
✅ Express.js ইনস্টল ও সেটআপ করা
✅ প্রথম সার্ভার তৈরি করা
✅ রাউট ও middleware ব্যবহার শেখা
✅ Nodemon দিয়ে কোড অটো-রিফ্রেশ করা

---

## 🎯 **Next Step (পরবর্তী ধাপ)**

👉 **Routing in Express.js**
👉 **Middleware & Static Files**
👉 **REST API (GET, POST, PUT, DELETE)**

---
