
---

# 🧠 1️⃣ What is Middleware (Simple Definition)

In **Express.js**,
**Middleware** means —
➡️ “Functions that run *between* the request coming from the client and the response going back to the client.”

They can:

* Access and modify **request (`req`)** and **response (`res`)** objects.
* Run any code (logging, validation, authentication, etc.).
* End the request–response cycle.
* Or call `next()` to move to the next middleware in the chain.

---

# ⚙️ 2️⃣ How Middleware Works (Internally)

When a client sends a request → Express processes it like this:

```
Client → [Middleware 1] → [Middleware 2] → [Middleware 3] → Route Handler → Response
```

👉 Each middleware gets three main objects:

```js
(req, res, next)
```

* **req:** the incoming request data (headers, params, body, etc.)
* **res:** the outgoing response object (used to send data back)
* **next:** a function that tells Express to move on to the next middleware.

If you don’t call `next()`, the request will **stop** right there — it won’t reach the next middleware or route handler.

---

# 🧩 3️⃣ Basic Structure of a Middleware Function

```js
function middlewareName(req, res, next) {
  // your code logic here
  console.log('Middleware running...');
  next(); // moves to next middleware or route
}
```

You “register” (attach) it in your app like this:

```js
const express = require('express');
const app = express();

app.use(middlewareName); // apply globally
```

---

# 🧱 4️⃣ Types of Middleware in Express.js

There are **five main types** 👇

| Type                  | Description                              | Example                 |
| --------------------- | ---------------------------------------- | ----------------------- |
| **Application-level** | Runs for all routes or specific routes.  | `app.use()`             |
| **Router-level**      | Runs only inside a specific router file. | `router.use()`          |
| **Built-in**          | Middleware that comes with Express.      | `express.json()`        |
| **Third-party**       | Installed from npm packages.             | `morgan`, `cors`, etc.  |
| **Error-handling**    | Catches and handles errors.              | `(err, req, res, next)` |

---

# 🧭 5️⃣ Application-Level Middleware (Global Middleware)

### Example:

```js
const express = require('express');
const app = express();

// Custom logger middleware
function logger(req, res, next) {
  console.log(`${req.method} ${req.url}`);
  next(); // move to next
}

app.use(logger); // applied to all routes

app.get('/', (req, res) => {
  res.send('Home Page');
});

app.get('/about', (req, res) => {
  res.send('About Page');
});

app.listen(3000);
```

🧠 Output in console:

```
GET /
GET /about
```

---

# 🧭 6️⃣ Router-Level Middleware

You can create a **Router** for specific parts of your app and attach middleware to that router only.

```js
// routes/admin.js
const express = require('express');
const router = express.Router();

// middleware only for admin routes
function checkAdmin(req, res, next) {
  const isAdmin = true; // assume logic here
  if (!isAdmin) return res.status(403).send('Forbidden');
  next();
}

router.use(checkAdmin); // applies only inside admin routes

router.get('/dashboard', (req, res) => {
  res.send('Welcome Admin!');
});

module.exports = router;

// main app
const app = express();
const adminRoutes = require('./routes/admin');
app.use('/admin', adminRoutes);
```

---

# ⚙️ 7️⃣ Built-in Middleware (comes with Express)

Express already has several ready-to-use middleware functions.

| Middleware                 | Purpose                                                     |
| -------------------------- | ----------------------------------------------------------- |
| `express.json()`           | Parses incoming JSON requests (makes `req.body` available). |
| `express.urlencoded()`     | Parses URL-encoded data (e.g. from forms).                  |
| `express.static('folder')` | Serves static files like images, CSS, JS.                   |

### Example:

```js
app.use(express.json());
app.use(express.static('public'));
app.use(express.urlencoded({ extended: true }));
```

---

# 🧰 8️⃣ Third-Party Middleware

These are npm packages that add extra features.

| Middleware        | Purpose                                |
| ----------------- | -------------------------------------- |
| `morgan`          | Logs request info.                     |
| `cors`            | Enables Cross-Origin Resource Sharing. |
| `cookie-parser`   | Parses cookies from request headers.   |
| `express-session` | Handles user sessions.                 |
| `helmet`          | Adds security headers.                 |

### Example:

```bash
npm install morgan cors
```

```js
const morgan = require('morgan');
const cors = require('cors');

app.use(morgan('dev'));
app.use(cors());
```

---

# ⚠️ 9️⃣ Error-Handling Middleware

Used to catch and respond to errors.

> Error-handling middleware **must have four parameters** `(err, req, res, next)`.

### Example:

```js
function errorHandler(err, req, res, next) {
  console.error(err.message);
  res.status(500).json({ error: 'Something went wrong!' });
}

app.use(errorHandler);
```

If you throw or pass an error with `next(err)`, it goes straight to this middleware.

```js
app.get('/crash', (req, res, next) => {
  try {
    throw new Error('Server broke!');
  } catch (err) {
    next(err);
  }
});
```

---

# 🔁 🔟 Middleware Execution Flow (Order Matters)

Express runs middleware in the **order you register them**.

### Example:

```js
app.use(first);
app.use(second);
app.get('/', (req, res) => res.send('done!'));
```

Execution order:

```
→ first
→ second
→ route handler
```

If `first` doesn’t call `next()`, then `second` and the route won’t run.

---

# 🧠 11️⃣ Real-Life Middleware Use Cases

| Use Case            | Description                                          |
| ------------------- | ---------------------------------------------------- |
| **Logging**         | Record who requested what (debugging, analytics).    |
| **Authentication**  | Check login token or session before allowing access. |
| **Data Validation** | Verify form data before inserting into database.     |
| **Security**        | Add headers or filters (Helmet).                     |
| **Compression**     | Reduce response size for performance.                |
| **Error Tracking**  | Centralize app errors in one middleware.             |

---

# 💡 12️⃣ Example — Multiple Middleware Chain

```js
function auth(req, res, next) {
  if (!req.headers.token) return res.status(401).send('Unauthorized');
  next();
}

function logRequest(req, res, next) {
  console.log(`${req.method} - ${req.url}`);
  next();
}

app.use(logRequest); // for all routes
app.get('/private', auth, (req, res) => res.send('Welcome authorized user!'));
```

👉 Here:

* `logRequest` runs for every route.
* `auth` runs only for `/private`.

---

# ⚡ 13️⃣ Async Middleware Example

When using async/await inside middleware, you need to handle rejected promises correctly.

```js
app.use(async (req, res, next) => {
  try {
    const user = await getUserFromDB();
    req.user = user;
    next();
  } catch (err) {
    next(err); // passes to error handler
  }
});
```

---

# 🧾 14️⃣ Best Practices for Middleware

✅ **Always call `next()`** unless you send a response.
✅ **Keep each middleware focused** (single responsibility).
✅ **Order matters:** register global middlewares first.
✅ **Handle errors properly:** never crash the server.
✅ **Don’t block the event loop:** use async I/O.
✅ **Place error handlers last.**

---

# 🔍 15️⃣ Summary (Quick Revision)

| Concept       | Summary                              |
| ------------- | ------------------------------------ |
| Middleware    | Function between request & response. |
| Key args      | `(req, res, next)`                   |
| Must call     | `next()`                             |
| Built-in      | `express.json()`, `express.static()` |
| Custom        | `app.use(myFunction)`                |
| Error handler | `(err, req, res, next)`              |
| Order         | Top → Bottom                         |
| Use cases     | Auth, logging, parsing, etc.         |

---
