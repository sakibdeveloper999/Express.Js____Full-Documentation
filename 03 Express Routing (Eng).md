Routing in Express is one of the first things to master for building web apps with Node. I’ll walk you through it step-by-step with clear examples and best practices so you can start routing confidently.

# 1. What is routing in Express?

Routing = matching an incoming HTTP request (method + URL) to some handler code that builds a response. Example: `GET /users/12` → a handler that returns user #12.

# 2. Minimal Express app with a route

```js
// index.js
import express from "express"; // or: const express = require('express')
const app = express();
const PORT = 3000;

// simple route
app.get("/", (req, res) => {
  res.send("Hello from Express!");
});

app.listen(PORT, () => console.log(`Server listening on ${PORT}`));
```

Run `node index.js` and visit `http://localhost:3000/`.

# 3. HTTP verbs / methods

Common method helpers:

* `app.get(path, handler)` — read
* `app.post(path, handler)` — create
* `app.put(path, handler)` / `app.patch(path, handler)` — update
* `app.delete(path, handler)` — delete
  Handlers receive `(req, res, next)`.

Example:

```js
app.post("/users", (req, res) => { /* create user */ });
app.put("/users/:id", (req, res) => { /* update user id */ });
app.delete("/users/:id", (req, res) => { /* delete user */ });
```

# 4. Route parameters and query strings

* Route params: `/users/:id` → `req.params.id`
* Query string: `/search?q=node&sort=desc` → `req.query.q`, `req.query.sort`

Example:

```js
app.get("/users/:id", (req, res) => {
  const id = req.params.id;
  const verbose = req.query.verbose === "true";
  res.json({ id, verbose });
});
```

# 5. Middleware basics (step by step)

Middleware is a function that runs before (or after) handlers. Signature: `(req, res, next)`.

* Application-level:

```js
app.use(express.json()); // built-in body parser
app.use((req, res, next) => {
  console.log(`${req.method} ${req.path}`);
  next(); // pass control
});
```

* Route-level:

```js
function requireAuth(req, res, next) {
  if (!req.headers.authorization) return res.status(401).send("Unauthorized");
  next();
}
app.get("/private", requireAuth, (req, res) => res.send("Welcome"));
```

# 6. Using `express.Router()` (modular routes)

Group related routes into a router—cleaner & mountable.

`routes/users.js`

```js
import express from "express";
const router = express.Router();

router.get("/", (req, res) => { /* list users */ });
router.post("/", (req, res) => { /* create user */ });
router.get("/:id", (req, res) => { /* get single user */ });

export default router;
```

Mount in main app:

```js
import usersRouter from "./routes/users.js";
app.use("/users", usersRouter);
// requests to /users, /users/:id handled by router
```

# 7. Route order matters

Express checks routes in the order they are declared. Put more specific routes before less specific (and `app.use` catch-alls later).

Bad:

```js
app.use("/users/:id", handlerA);
app.get("/users/me", handlerB); // never reached if /users/:id was placed first
```

Good:

```js
app.get("/users/me", handlerB);
app.get("/users/:id", handlerA);
```

# 8. Route chaining and multiple handlers

You can attach several handlers to a route:

```js
app.get(
  "/files/:id",
  middleware1,
  middleware2,
  (req, res) => { res.send("done"); }
);
```

Or use `router.route()` to chain methods for same path:

```js
router.route("/:id")
  .get(getUser)
  .put(updateUser)
  .delete(deleteUser);
```

# 9. Async route handlers & error handling

When using `async/await`, catch errors and call `next(err)` (or use a helper to wrap async handlers).

Simple wrapper:

```js
const wrap = (fn) => (req, res, next) => fn(req, res, next).catch(next);

router.get("/:id", wrap(async (req, res) => {
  const user = await db.findUser(req.params.id);
  res.json(user);
}));
```

Error-handling middleware:

```js
app.use((err, req, res, next) => {
  console.error(err);
  res.status(500).json({ error: "Internal Server Error" });
});
```

Place error handler after all routes.

# 10. Static files & default routing

Serve public/static files:

```js
app.use(express.static("public")); // serves /public/index.html at /
```

If building an SPA, you may need to fall back to `index.html` for unmatched GET requests (put after API routes).

# 11. Route protection / auth patterns

Common pattern: middleware that verifies JWT/session and attaches `req.user`.

```js
function auth(req, res, next) {
  // verify token, set req.user
  if (!valid) return res.status(401).send("Unauthorized");
  next();
}
app.use("/api", auth, apiRouter); // protects all /api routes
```

# 12. Validation & sanitization

Validate body/params before handlers (e.g., with libraries like `express-validator` or `zod`). Example pattern:

```js
router.post("/", validateCreateUser, (req, res) => { /* req.body is valid */ });
```

# 13. Organizing route files (recommended project structure)

```
project/
├─ index.js               // app, middleware, mount routers
├─ routes/
│  ├─ users.js
│  └─ auth.js
├─ controllers/
│  ├─ usersController.js
│  └─ authController.js
├─ middlewares/
│  └─ auth.js
├─ services/
└─ models/
```

Keep handlers small — controllers call services (business logic).

# 14. Debugging tips & common gotchas

* Forgetting `next()` or not returning after sending a response → request may hang or error.
* Route order: specific before general.
* Using `app.use()` without path mounts at `/` and may override other routes.
* Not parsing body (`express.json()`) before reading `req.body`.
* Mixing `res.send()` and `res.json()` incorrectly — either works, but beware repeated sends.

# 15. Example: small CRUD API

```js
// index.js
import express from "express";
import usersRouter from "./routes/users.js";

const app = express();
app.use(express.json());
app.use("/users", usersRouter);

app.use((err, req, res, next) => {
  console.error(err.stack);
  res.status(500).json({ message: "Something blew up" });
});

app.listen(3000);
```

```js
// routes/users.js
import express from "express";
const router = express.Router();

let users = [{ id: "1", name: "Alice" }];

router.get("/", (req, res) => res.json(users));
router.post("/", (req, res) => {
  const user = { id: String(Date.now()), ...req.body };
  users.push(user);
  res.status(201).json(user);
});
router.get("/:id", (req, res) => {
  const u = users.find(x => x.id === req.params.id);
  if (!u) return res.status(404).send("Not found");
  res.json(u);
});
router.put("/:id", (req, res) => {
  const idx = users.findIndex(x => x.id === req.params.id);
  if (idx === -1) return res.status(404).send("Not found");
  users[idx] = { ...users[idx], ...req.body };
  res.json(users[idx]);
});
router.delete("/:id", (req, res) => {
  users = users.filter(x => x.id !== req.params.id);
  res.status(204).end();
});

export default router;
```

# 16. Best practices (short checklist)

* Use `express.json()` and `express.urlencoded()` early if you need request bodies.
* Keep routes organized with `express.Router()`.
* Validate inputs before doing work.
* Centralize error handling with an error middleware.
* Protect routes with auth middleware.
* Keep route handlers thin: controllers → services → DB.
* Use meaningful HTTP status codes (201 on creation, 204 for delete no content, 400 for bad request, 404 not found, 401/403 for auth).

# 17. Quick reference

* `req.params` — route parameters
* `req.query` — query string
* `req.body` — parsed body (needs body parser)
* `res.status(code).json(obj)` — set status + send JSON
* `next(err)` — pass error to error middleware

---
