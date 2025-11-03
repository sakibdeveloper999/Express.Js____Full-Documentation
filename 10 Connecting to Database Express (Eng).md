— let’s walk through **Connecting to a Database in Express.js** step-by-step. I’ll cover concepts, recommended patterns, 2 concrete examples (Postgres using `pg` + connection pool, and MongoDB using `mongoose`), file layouts, CRUD endpoints, transactions, environment/config, and production tips. Examples use **ES modules** (import/export) and async/await.

---

# 1) High-level overview — what “connect to a DB” means

1. Install a driver/ORM for the database you choose (Postgres, MySQL, MongoDB, etc.).
2. Configure a connection string and credentials (keep them out of source control).
3. Open a connection (or pool) from your app to the DB.
4. Use that connection to run queries or ORM calls in your request handlers/services.
5. Handle errors, timeouts, graceful shutdown, and pooling for production.

# 2) Choose approach: driver vs query builder vs ORM

* **Driver** (e.g., `pg`, `mysql2`, `mongodb`): low-level, most control, minimal abstraction — good for simple queries and performance-critical code.
* **Query builder** (e.g., `knex`): safer SQL construction, migrations, cross-db capability.
* **ORM** (e.g., `Sequelize`, `TypeORM`, `Mongoose` for MongoDB): maps DB objects to JavaScript classes; speeds development for complex models but can hide SQL details and performance costs.

# 3) Best practices (short)

* Store DB credentials in environment variables (e.g., `.env` + `dotenv`).
* Use connection pooling for RDBMS and manage pool size.
* Use parameterized queries / prepared statements (avoid SQL injection).
* Use transactions for multi-step updates that must be atomic.
* Gracefully close DB connections on process exit.
* Keep DB logic out of route handlers — use a service/data-access layer.
* Use migrations (e.g., `knex migrate`, `sequelize-cli`) for schema changes.
* Limit the number of concurrent DB connections (use pg-bouncer for Postgres with many app instances).

---

# 4) Small project structure (example)

```
my-express-app/
├─ package.json
├─ .env
├─ src/
│  ├─ index.mjs           # server entry
│  ├─ app.mjs             # express app + middleware
│  ├─ db/
│  │  ├─ pgPool.mjs       # Postgres pool (or mongoose.js)
│  ├─ routes/
│  │  └─ users.js
│  ├─ controllers/
│  │  └─ usersController.js
│  └─ services/
│     └─ usersService.js
```

---

# 5) Example A — Postgres using `pg` (connection pool, parameterized queries)

## Install

```
npm install express pg dotenv
```

## .env

```
DATABASE_URL=postgres://username:password@localhost:5432/mydb
PORT=3000
```

## src/db/pgPool.mjs

```js
import pg from "pg";
import dotenv from "dotenv";
dotenv.config();

const { Pool } = pg;
const pool = new Pool({
  connectionString: process.env.DATABASE_URL,
  // optional tuning
  max: 20,
  idleTimeoutMillis: 30000,
  connectionTimeoutMillis: 2000,
});

pool.on("error", (err) => {
  console.error("Unexpected pg pool error", err);
  // optionally process.exit(1);
});

export default pool;
```

## src/services/usersService.js

```js
import pool from "../db/pgPool.mjs";

// get all users
export async function getAllUsers() {
  const { rows } = await pool.query("SELECT id, name, email FROM users ORDER BY id");
  return rows;
}

// get user by id (parameterized)
export async function getUserById(id) {
  const { rows } = await pool.query("SELECT id, name, email FROM users WHERE id = $1", [id]);
  return rows[0] ?? null;
}

// create user (returns created row)
export async function createUser({ name, email }) {
  const { rows } = await pool.query(
    "INSERT INTO users (name, email) VALUES ($1, $2) RETURNING id, name, email",
    [name, email]
  );
  return rows[0];
}
```

## src/routes/users.js

```js
import express from "express";
import * as usersService from "../services/usersService.js";
const router = express.Router();

router.get("/", async (req, res, next) => {
  try {
    const users = await usersService.getAllUsers();
    res.json(users);
  } catch (err) { next(err); }
});

router.get("/:id", async (req, res, next) => {
  try {
    const user = await usersService.getUserById(req.params.id);
    if (!user) return res.status(404).json({ message: "Not found" });
    res.json(user);
  } catch (err) { next(err); }
});

router.post("/", async (req, res, next) => {
  try {
    const newUser = await usersService.createUser(req.body);
    res.status(201).json(newUser);
  } catch (err) { next(err); }
});

export default router;
```

## src/index.mjs (entry)

```js
import express from "express";
import dotenv from "dotenv";
import usersRouter from "./routes/users.js";
import pool from "./db/pgPool.mjs";

dotenv.config();
const app = express();
app.use(express.json());

app.use("/users", usersRouter);

app.use((err, req, res, next) => {
  console.error(err);
  res.status(500).json({ error: "Internal server error" });
});

const server = app.listen(process.env.PORT || 3000, () => {
  console.log("Server started");
});

// Graceful shutdown: close pool and server
process.on("SIGINT", async () => {
  console.log("SIGINT — shutting down");
  await pool.end(); // closes pg pool
  server.close(() => process.exit(0));
});
```

### Transaction example (pg)

```js
// in some service
export async function transferMoney(fromId, toId, amount) {
  const client = await pool.connect();
  try {
    await client.query("BEGIN");
    await client.query("UPDATE accounts SET balance = balance - $1 WHERE id = $2", [amount, fromId]);
    await client.query("UPDATE accounts SET balance = balance + $1 WHERE id = $2", [amount, toId]);
    await client.query("COMMIT");
  } catch (err) {
    await client.query("ROLLBACK");
    throw err;
  } finally {
    client.release();
  }
}
```

---

# 6) Example B — MongoDB using `mongoose`

## Install

```
npm install express mongoose dotenv
```

## .env

```
MONGO_URI=mongodb://localhost:27017/mydb
PORT=3000
```

## src/db/mongoose.mjs

```js
import mongoose from "mongoose";
import dotenv from "dotenv";
dotenv.config();

const uri = process.env.MONGO_URI;

mongoose.set("strictQuery", true);

export async function connectMongoose() {
  try {
    await mongoose.connect(uri, {
      // options as needed
      useNewUrlParser: true,
      useUnifiedTopology: true,
    });
    console.log("Connected to MongoDB");
  } catch (err) {
    console.error("MongoDB connection error:", err);
    throw err;
  }
}

export function disconnectMongoose() {
  return mongoose.disconnect();
}
```

## src/models/User.mjs

```js
import mongoose from "mongoose";
const { Schema, model } = mongoose;

const userSchema = new Schema({
  name: { type: String, required: true },
  email: { type: String, required: true, unique: true, index: true },
}, { timestamps: true });

export default model("User", userSchema);
```

## src/services/usersService.js (mongoose)

```js
import User from "../models/User.mjs";

export const getAllUsers = () => User.find().select("name email").lean();
export const getUserById = (id) => User.findById(id).select("name email").lean();
export const createUser = (data) => User.create(data);
```

## src/index.mjs

```js
import express from "express";
import dotenv from "dotenv";
import { connectMongoose, disconnectMongoose } from "./db/mongoose.mjs";
import usersRouter from "./routes/users.js";

dotenv.config();
const app = express();
app.use(express.json());

app.use("/users", usersRouter);

const server = app.listen(process.env.PORT || 3000, async () => {
  await connectMongoose();
  console.log("Server started");
});

process.on("SIGINT", async () => {
  console.log("SIGINT — shutting down");
  await disconnectMongoose();
  server.close(() => process.exit(0));
});
```

---

# 7) Where to put DB logic

* Keep SQL/DB code inside a `services` or `repositories` layer (e.g., `services/usersService.js`).
* Controllers handle request/response only and call service functions.
* This improves testability and separation of concerns.

---

# 8) Simple CRUD flow illustrated

1. Client → `POST /users` with `{ name, email }`.
2. Route receives request and calls `createUser` service.
3. Service uses DB pool or ORM to insert row / document.
4. Service returns created entity to controller → response `201`.

---

# 9) Common errors & troubleshooting

* `ECONNREFUSED` — DB server not running / wrong host/port.
* `authentication failed` — wrong username/password or connection string.
* `too many clients` — pool max too high or too many app instances; use connection pooler (pg-bouncer).
* Mongo unique index duplicate error — ensure unique index exists and handle `E11000` errors.
* Long queries/timeouts — add query timeouts, indexes, or optimize SQL.

---

# 10) Testing tips

* Use a test DB or in-memory DB for tests (e.g., SQLite for SQL or `mongodb-memory-server` for Mongo).
* Run migrations in CI before tests.
* Mock DB layer for unit tests (mock `services` functions).

---

# 11) Production considerations

* Use SSL/TLS for DB connections when connecting to managed DBs.
* Rotate credentials and use secrets manager (AWS Secrets Manager, Vault).
* Use read replicas for scaling reads; use a leader for writes.
* Use connection poolers (PgBouncer) for serverless / many ephemeral connections.
* Monitor query performance, slow queries, connection usage.

---

# 12) Quick checklist to get a DB-connected Express app running

1. Choose DB and approach (pg + SQL, mongoose + MongoDB, ORM).
2. Install driver/ORM packages.
3. Add `.env` with DB connection string.
4. Create `db/` module to initialize pool/connection and export it.
5. Write a small `services` layer for DB access (parameterized queries).
6. Wire routes to services and test basic CRUD endpoints with Postman/curl.
7. Add error handling and graceful shutdown.
8. Add migrations and tests.
9. Tune pool sizes and connection settings for production.

---

