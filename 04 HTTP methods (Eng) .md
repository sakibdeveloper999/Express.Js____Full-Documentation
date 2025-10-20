# One-sentence summary

HTTP methods in Express.js (GET, POST, PUT, PATCH, DELETE, HEAD, OPTIONS, etc.) map HTTP intent to route handlers — this guide walks you step-by-step through how each method works, how to use them correctly, how to structure routes and middleware, and real-world practices, examples and pitfalls.

---

# 1) Quick prerequisite — create the starter app

```bash
mkdir express-http-methods-detailed
cd express-http-methods-detailed
npm init -y
npm i express
```

Create `app.js`:

```js
const express = require('express');
const app = express();
const port = 3000;

app.use(express.json());               // parse application/json
app.use(express.urlencoded({ extended: true })); // parse form bodies

app.get('/', (req, res) => res.send('Hello Express'));
app.listen(port, () => console.log(`Server listening http://localhost:${port}`));
```

Run with: `node app.js`.

---

# 2) HTTP methods explained (semantics + when to use)

* **GET** — *Retrieve a representation of a resource.*

  * Should be **safe** (no side effects) and **idempotent**.
  * Use for fetching lists, single items, search results.
  * Supports query strings (`req.query`) and route params (`req.params`).

* **HEAD** — *Same as GET but without response body.*

  * Useful for checking headers (content-length, caching) or health checks.
  * Express will automatically handle HEAD if you define GET for the same route.

* **POST** — *Create a new resource or perform an action.*

  * Not idempotent by default (repeating may create duplicates).
  * Body typically contains new resource data (`req.body`).

* **PUT** — *Replace a resource.*

  * Idempotent: same request repeated yields same state.
  * Commonly used to replace whole resource (all fields).

* **PATCH** — *Partial update of a resource.*

  * Used for changing a subset of fields. If implemented carefully it can be idempotent.

* **DELETE** — *Delete a resource.*

  * Should be idempotent (deleting already-deleted resource is usually handled gracefully).

* **OPTIONS** — *Describe communication options for a resource.*

  * Used for CORS preflight requests and to advertise allowed methods (Allow header).

**Key principle:** pick the method that conveys the intended action. Avoid using GET for changes.

---

# 3) Route basics and how to declare handlers

### Simple handlers

```js
app.get('/items', (req, res) => { /* list items */ });
app.post('/items', (req, res) => { /* create item */ });
app.get('/items/:id', (req, res) => { /* get item */ });
```

### Route chaining with `app.route`

```js
app.route('/items/:id')
  .get(getItem)
  .put(replaceItem)
  .patch(updateItem)
  .delete(deleteItem);
```

### Modular routes with Router

```js
// routes/users.js
const express = require('express');
const router = express.Router();

router.get('/', listUsers);
router.post('/', createUser);

module.exports = router;

// app.js
const usersRouter = require('./routes/users');
app.use('/users', usersRouter);
```

---

# 4) Accessing request data (params, query, body, headers, files)

* `req.params` → route params (`/users/:id`).
* `req.query` → query string (`/search?q=term&page=2`).
* `req.body` → request body (needs `express.json()` or `express.urlencoded()`).
* `req.headers` → request headers.
* `req.get('Header-Name')` → convenience to read a header.
* Files: use `multer` to parse `multipart/form-data` uploads.

Example: using `multer`:

```js
const multer = require('multer');
const upload = multer({ dest: 'uploads/' });

app.post('/profile', upload.single('avatar'), (req, res) => {
  // req.file and req.body available
  res.json({ file: req.file, body: req.body });
});
```

---

# 5) Example: full in-memory CRUD service (step-by-step)

Create `users.js` as a small app to experiment:

```js
const express = require('express');
const app = express();
app.use(express.json());

let users = [];
let nextId = 1;

// LIST (GET /users)
app.get('/users', (req, res) => {
  // support filtering with query: /users?name=alice
  const { name } = req.query;
  const results = name ? users.filter(u => u.name.includes(name)) : users;
  res.json(results);
});

// CREATE (POST /users)
app.post('/users', (req, res) => {
  const { name, age } = req.body;
  if (!name) return res.status(400).json({ error: 'name required' });
  const user = { id: nextId++, name, age };
  users.push(user);
  res.status(201).json(user);     // 201 Created
});

// READ (GET /users/:id)
app.get('/users/:id', (req, res) => {
  const id = Number(req.params.id);
  const user = users.find(u => u.id === id);
  if (!user) return res.status(404).json({ error: 'not found' });
  res.json(user);
});

// REPLACE (PUT /users/:id)
app.put('/users/:id', (req, res) => {
  const id = Number(req.params.id);
  const idx = users.findIndex(u => u.id === id);
  if (idx === -1) return res.status(404).json({ error: 'not found' });
  const { name, age } = req.body;
  if (!name) return res.status(400).json({ error: 'name required' });
  users[idx] = { id, name, age };
  res.json(users[idx]);
});

// PATCH (PATCH /users/:id)
app.patch('/users/:id', (req, res) => {
  const id = Number(req.params.id);
  const user = users.find(u => u.id === id);
  if (!user) return res.status(404).json({ error: 'not found' });
  Object.assign(user, req.body);
  res.json(user);
});

// DELETE (DELETE /users/:id)
app.delete('/users/:id', (req, res) => {
  const id = Number(req.params.id);
  users = users.filter(u => u.id !== id);
  res.sendStatus(204); // No Content
});

app.listen(3000);
```

Test with `curl` or Postman (examples below).

---

# 6) Example curl commands (cheat-sheet)

* Create:

```bash
curl -X POST -H "Content-Type: application/json" -d '{"name":"Alice"}' http://localhost:3000/users
```

* List:

```bash
curl http://localhost:3000/users
```

* Get single:

```bash
curl http://localhost:3000/users/1
```

* Replace:

```bash
curl -X PUT -H "Content-Type: application/json" -d '{"name":"Alicia","age":30}' http://localhost:3000/users/1
```

* Patch:

```bash
curl -X PATCH -H "Content-Type: application/json" -d '{"age":31}' http://localhost:3000/users/1
```

* Delete:

```bash
curl -X DELETE http://localhost:3000/users/1
```

---

# 7) Status codes — proper use & why they matter

* `200 OK` — standard success (GET).
* `201 Created` — resource created (POST). Include `Location` header pointing to new resource when possible.
* `204 No Content` — success with no body (DELETE, sometimes PUT).
* `400 Bad Request` — validation or malformed request.
* `401 Unauthorized` — unauthenticated (user must login).
* `403 Forbidden` — authenticated but not allowed.
* `404 Not Found` — resource does not exist.
* `409 Conflict` — resource conflict (duplicate unique field).
* `422 Unprocessable Entity` — semantic validation failed (optional).
* `500 Internal Server Error` — unexpected server error.

Always return meaningful JSON for errors in APIs (avoid HTML error pages for JSON APIs).

---

# 8) Error handling patterns in Express

### Synchronous errors: throw or next(err)

```js
app.get('/sync', (req, res) => {
  throw new Error('boom'); // gets handled by error middleware
});
```

### Async errors: use next or a wrapper

```js
app.get('/async', async (req, res, next) => {
  try {
    await doSomethingAsync();
    res.send('done');
  } catch (err) {
    next(err);
  }
});

// Promise wrapper to avoid repeating try/catch
const wrap = (fn) => (req, res, next) => Promise.resolve(fn(req, res, next)).catch(next);
app.get('/async2', wrap(async (req, res) => { await ... }));
```

### Centralized error middleware (last middleware)

```js
app.use((err, req, res, next) => {
  console.error(err);
  const status = err.status || 500;
  res.status(status).json({ error: err.message || 'Internal Server Error' });
});
```

---

# 9) Middleware ordering and best practices

* Middleware registration order matters. Register body parsers and auth middleware **before** routes that need them.
* Example order:

  1. Security middleware (helmet)
  2. Logging (morgan)
  3. Body parsers
  4. CORS
  5. Authentication
  6. Routes
  7. Error handler

```js
app.use(require('helmet')());
app.use(require('morgan')('combined'));
app.use(express.json());
app.use(require('cors')());
app.use(authMiddleware);    // if common auth
app.use('/api', apiRouter);
app.use((err, req, res, next) => { /* error handler */ });
```

---

# 10) Validation & typing

* Validate inputs to avoid bad data. Options:

  * `express-validator` (middleware-style).
  * `Joi` or `Zod` for schemas (validate in handlers).
  * TypeScript for compile-time types + runtime checks (still need runtime validation).

Example with `express-validator`:

```js
const { body, validationResult } = require('express-validator');

app.post('/users', [
  body('name').isString().notEmpty(),
  body('age').optional().isInt({ min: 0 })
], (req, res) => {
  const errors = validationResult(req);
  if (!errors.isEmpty()) return res.status(400).json({ errors: errors.array() });
  // proceed
});
```

---

# 11) CORS, OPTIONS and preflight requests

* Browsers send OPTIONS preflight when cross-origin requests use non-simple methods/headers. Use `cors` package:

```js
const cors = require('cors');
app.use(cors({ origin: 'https://example.com' }));
```

* You can also add an OPTIONS handler manually, but using `cors()` is simpler and safer.

---

# 12) Caching, ETag, and conditional requests

* Use `res.set('Cache-Control', 'public, max-age=3600')` to direct caches.
* Express will compute ETag automatically for many responses. Clients can send `If-None-Match` and server can return `304 Not Modified`.
* Use caching wisely for GET requests.

---

# 13) Idempotency & safety practical notes

* **Safe** methods (no side effects): GET, HEAD, OPTIONS.
* **Idempotent** methods (repeated calls produce same effect): GET, PUT, DELETE, HEAD, OPTIONS. POST is typically not idempotent.
* For operations like payment or order submission, use idempotency keys (e.g., client sends `Idempotency-Key` header) to avoid duplicate processing if the client retries.

---

# 14) File uploads & streaming

* For file uploads use `multer` for typical uploads or streams for large files.
* For streaming large responses (e.g., file download), use `res.download()` or stream from file system/DB to response to avoid large memory footprint:

```js
const fs = require('fs');
app.get('/download/:file', (req, res) => {
  const stream = fs.createReadStream(`/data/${req.params.file}`);
  stream.pipe(res);
});
```

---

# 15) Security best practices for HTTP methods

* Use HTTPS — never serve sensitive APIs over HTTP in production.
* Use `helmet()` to set secure headers.
* Validate and sanitize inputs to prevent injection attacks.
* Use proper authentication/authorization (JWT, OAuth, sessions). Don't use GET for state-changing actions (no side effects).
* Rate limit endpoints (`express-rate-limit`) to prevent brute force and DoS.
* When returning resources, avoid leaking sensitive fields (e.g., passwords, tokens).

---

# 16) Testing your HTTP methods

* Unit/integration tests with `supertest` + `jest` / `mocha`:

```js
const request = require('supertest');
const app = require('./app'); // your express app

test('create user', async () => {
  const res = await request(app)
    .post('/users')
    .send({ name: 'Alice' })
    .expect(201);
  expect(res.body).toHaveProperty('id');
});
```

* Postman or Insomnia are good for manual exploratory testing.

---

# 17) Real-world API design tips (RESTful considerations)

* Use nouns not verbs in URLs: `/orders` not `/createOrder`.
* Use nested resources sparingly: `/users/:id/orders`.
* Use plural nouns: `/users`, `/products`.
* Return consistent response shapes: `{ data: ..., error: ... }` or `{ success: true, payload: ... }` (pick one and be consistent).
* Use HATEOAS only if you specifically need hypermedia (most APIs don't).
* Version your API: `/v1/users` to allow safe evolution.

---

# 18) Troubleshooting common problems

* `req.body` is undefined → forgot `app.use(express.json())`.
* `CORS` errors in browser → configure `cors()` properly or allow the origin.
* Asynchronous errors not handled → wrap async handlers or use a wrapper so errors reach error middleware.
* Wrong status codes → follow REST conventions above.
* Routes not matching → check order: more specific routes should come before generic wildcard routes.

---

# 19) Advanced: conditional requests, PATCH semantics, and hypermedia

* Implement PATCH with JSON Merge Patch or JSON Patch (RFC 7386 / RFC 6902) for clear partial update semantics.
* Support conditional PATCH/PUT with `If-Match` header and ETags to prevent lost updates (optimistic concurrency):

  * Client gets ETag of resource.
  * When updating, client sends `If-Match: "<etag>"`.
  * Server responds `412 Precondition Failed` if resource changed.

---

# 20) Example project structure for medium-sized API

```
/project
  /src
    /routes
      users.js
      products.js
    /controllers
      users.js
    /services
      usersService.js
    /middlewares
      auth.js
      errorHandler.js
    app.js
  package.json
```

* Controllers implement route logic, services handle business logic/data store access, middlewares handle cross-cutting concerns.

---

# 21) Helpful middleware & libraries

* `helmet` — security headers
* `cors` — CORS handling
* `morgan` — logging
* `express-rate-limit` — rate limiting
* `multer` — file uploads
* `express-validator`, `joi`, `zod` — validation
* `winston` or `pino` — production logging
* `supertest` + `jest` — testing

---

# 22) Practical checklist before production

* ✅ Use HTTPS and secure cookies.
* ✅ Add authentication & authorization.
* ✅ Validate inputs and output shapes.
* ✅ Limit payload sizes (`express.json({ limit: '1mb' })`).
* ✅ Set sane CORS policies.
* ✅ Add rate limits.
* ✅ Hide stack traces in production error responses.
* ✅ Log requests and errors consistently.
* ✅ Add monitoring/health-check endpoints.

---

# 23) Summary examples and quick decision rules

* Want to **fetch** → `GET`.
* Want to **create** → `POST` (respond `201 Created`, include `Location`).
* Want to **replace whole** entity → `PUT`.
* Want to **update partial** → `PATCH`.
* Want to **delete** → `DELETE` (respond `204 No Content` if nothing returned).
* Browser preflight → `OPTIONS` (use `cors()`).

---
