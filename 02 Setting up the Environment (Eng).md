
---

# 🚀 **Setting Up the Express.js Environment (Full Step-by-Step Guide)**

---

## 🧩 **1. What is Express.js?**

**Express.js** is a **web application framework** for **Node.js** that simplifies building web servers and APIs.
Instead of writing raw Node.js code (which is long), Express provides easy methods like `app.get()`, `app.post()`, and middleware support.

**Key features:**

* Fast and minimal setup
* Middleware support
* Easy routing
* Template engine integration
* Works perfectly for REST APIs

---

## ⚙️ **2. Prerequisites**

Before installing Express.js, make sure you have:

✅ **Node.js** and **npm** installed.

### Check installation:

```bash
node -v
npm -v
```

If not installed, download and install from:
👉 [https://nodejs.org](https://nodejs.org)

---

## 📁 **3. Create a New Project Folder**

Open your terminal (or VS Code terminal) and run:

```bash
mkdir expressjs-app
cd expressjs-app
```

This creates a new folder for your project and opens it.

---

## 📦 **4. Initialize a Node.js Project**

Inside your folder, initialize a Node project:

```bash
npm init -y
```

This command creates a `package.json` file that stores project info and dependencies.

Example `package.json` after running:

```json
{
  "name": "expressjs-app",
  "version": "1.0.0",
  "main": "index.js",
  "scripts": {
    "start": "node index.js"
  },
  "license": "ISC"
}
```

---

## 🌐 **5. Install Express.js**

Now, install Express.js as a dependency:

```bash
npm install express
```

✅ This will add Express to your `node_modules` folder
✅ And add a dependency entry in your `package.json` file.

---

## 🧠 **6. Create Your First Express App**

Create a file named `index.js` (or `app.js`) inside the root folder.

```bash
touch index.js
```

### ✍️ Add this code inside `index.js`:

```js
// Step 1: Import express
const express = require('express');

// Step 2: Create an express application
const app = express();

// Step 3: Define a route
app.get('/', (req, res) => {
  res.send('Hello, Express.js!');
});

// Step 4: Start the server
const PORT = 3000;
app.listen(PORT, () => {
  console.log(`Server running on http://localhost:${PORT}`);
});
```

---

## ▶️ **7. Run the Server**

Run your app using:

```bash
node index.js
```

Now open your browser and go to:
👉 [http://localhost:3000](http://localhost:3000)

You’ll see:

```
Hello, Express.js!
```

🎉 Congratulations — your Express.js environment is ready!

---

## 🔁 **8. Auto Restart using Nodemon (Optional but Recommended)**

When you edit your code, you currently have to stop and restart the server.
Let’s fix that with **nodemon**.

Install it globally:

```bash
npm install -g nodemon
```

Or as a dev dependency:

```bash
npm install --save-dev nodemon
```

Then update your `package.json`:

```json
"scripts": {
  "start": "node index.js",
  "dev": "nodemon index.js"
}
```

Now start your app using:

```bash
npm run dev
```

✅ Your server will automatically restart when you make changes.

---

## 🧩 **9. Folder Structure (Recommended)**

As your project grows, organize it like this:

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

---

## ⚡ **10. Add Middleware Example**

Middleware are functions that run between a request and a response.

Example:

```js
app.use(express.json()); // built-in middleware to parse JSON data

app.use((req, res, next) => {
  console.log(`${req.method} ${req.url}`);
  next();
});
```

---

## 🔄 **11. Add More Routes**

```js
app.get('/about', (req, res) => {
  res.send('About Page');
});

app.get('/contact', (req, res) => {
  res.send('Contact Page');
});
```

Now you can visit:

* [http://localhost:3000/](http://localhost:3000/)
* [http://localhost:3000/about](http://localhost:3000/about)
* [http://localhost:3000/contact](http://localhost:3000/contact)

---

## ✅ **12. Common Commands Summary**

| Command               | Description           |
| --------------------- | --------------------- |
| `npm init -y`         | Create `package.json` |
| `npm install express` | Install Express.js    |
| `node index.js`       | Run the server        |
| `npm run dev`         | Run with nodemon      |
| `ctrl + c`            | Stop the server       |

---

## 🧾 **13. Troubleshooting Tips**

| Problem                               | Solution                        |
| ------------------------------------- | ------------------------------- |
| `Error: Cannot find module 'express'` | Run `npm install express` again |
| `EADDRINUSE`                          | Change port number              |
| `Permission denied`                   | Try `sudo` or admin terminal    |
| JSON not parsing                      | Add `app.use(express.json())`   |

---

## 🧠 **14. What You Learned**

You can now:
✅ Set up Node.js & Express.js
✅ Create a simple web server
✅ Handle multiple routes
✅ Use middleware
✅ Use nodemon for auto restart

---

## 🎯 **Next Step (Coming Up)**

👉 **Routing in Express.js**
👉 **Middleware & Static Files**
👉 **REST API (GET, POST, PUT, DELETE)**

---

