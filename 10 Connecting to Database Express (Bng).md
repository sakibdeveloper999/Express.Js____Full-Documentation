
# 🧠 **Express.js এ Database Connection — সম্পূর্ণ ডকুমেন্টেশন (Bangla Version)**

---

## 🎯 ১. Database Connection মানে কী?

Express.js একটি **server-side web framework**, কিন্তু এটি **নিজে কোনো database handle করে না**।
তুমি যখন ডাটা সংরক্ষণ করতে চাও (যেমন user list, product list, orders, etc.), তখন তোমাকে কোনো database system (MySQL, PostgreSQL, MongoDB ইত্যাদি) ব্যবহার করতে হয়।

👉 তাই “**Connecting to a Database**” মানে হলো:

> Express.js অ্যাপ্লিকেশন থেকে ডাটাবেসের সাথে একটি স্থায়ী যোগাযোগ তৈরি করা (connection তৈরি করা) যাতে সার্ভার কোড থেকে query চালানো যায়, ডাটা insert/update/read/delete করা যায়।

---

## ⚙️ ২. Step by Step কাজের Flow (General Concept)

1. **Database Driver/ORM install করো**

   * যেমন: PostgreSQL এর জন্য `pg`, MongoDB এর জন্য `mongoose`
2. **Connection string সেট করো** (.env ফাইল এ রাখো)
3. **DB connection তৈরি করো** (pool বা mongoose.connect দিয়ে)
4. **Query চালাও** (select, insert, update, delete)
5. **Error handling ও graceful shutdown করো**

---

## 🧩 ৩. Database এর ধরন (SQL বনাম NoSQL)

| ধরন   | Database          | Library/Driver                      | বৈশিষ্ট্য                                      |
| ----- | ----------------- | ----------------------------------- | ---------------------------------------------- |
| SQL   | PostgreSQL, MySQL | `pg`, `mysql2`, `sequelize`, `knex` | Structured data, table, row-column format      |
| NoSQL | MongoDB           | `mongoose`, `mongodb`               | JSON-like flexible documents, no strict schema |

---

## 🧱 ৪. প্রজেক্ট স্ট্রাকচার (Common Structure)

```
my-express-app/
├─ package.json
├─ .env
├─ src/
│  ├─ index.mjs          # main entry point
│  ├─ app.mjs            # express app
│  ├─ db/                # DB connection files
│  │  ├─ pgPool.mjs      # Postgres connection
│  │  ├─ mongoose.mjs    # Mongo connection
│  ├─ models/            # schema/model (for Mongo)
│  ├─ services/          # data access logic
│  ├─ routes/            # API routes
│  └─ controllers/       # request handler logic
```

---

## 🧠 ৫. PostgreSQL ব্যবহার করা — `pg` লাইব্রেরি সহ (Full Setup)

### 🔹 Step 1: Install

```bash
npm install express pg dotenv
```

### 🔹 Step 2: .env ফাইল তৈরি করো

```
DATABASE_URL=postgres://username:password@localhost:5432/mydb
PORT=3000
```

### 🔹 Step 3: Connection setup — src/db/pgPool.mjs

```js
import pg from "pg";
import dotenv from "dotenv";
dotenv.config();

const { Pool } = pg;

const pool = new Pool({
  connectionString: process.env.DATABASE_URL,
  max: 20,
  idleTimeoutMillis: 30000,
  connectionTimeoutMillis: 2000,
});

pool.on("error", (err) => {
  console.error("Unexpected PG Pool error:", err);
});

export default pool;
```

🧩 **ব্যাখ্যা:**

* `Pool()` দিয়ে একটি **connection pool** তৈরি করা হয়েছে।
* এটি একাধিক connection manage করে।
* প্রতিবার নতুন connection না বানিয়ে pool থেকে reuse করে।

---

### 🔹 Step 4: Service Layer — src/services/usersService.js

```js
import pool from "../db/pgPool.mjs";

// সব user আনো
export async function getAllUsers() {
  const { rows } = await pool.query("SELECT id, name, email FROM users");
  return rows;
}

// নির্দিষ্ট user আনো
export async function getUserById(id) {
  const { rows } = await pool.query("SELECT * FROM users WHERE id = $1", [id]);
  return rows[0];
}

// নতুন user তৈরি করো
export async function createUser({ name, email }) {
  const { rows } = await pool.query(
    "INSERT INTO users (name, email) VALUES ($1, $2) RETURNING *",
    [name, email]
  );
  return rows[0];
}
```

🧩 **ব্যাখ্যা:**

* `$1`, `$2` হলো parameterized query placeholder → SQL injection প্রতিরোধ করে।
* pool.query() Promise return করে, তাই `await` ব্যবহার।

---

### 🔹 Step 5: Route Layer — src/routes/users.js

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
    if (!user) return res.status(404).json({ message: "User not found" });
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

---

### 🔹 Step 6: Main Entry — src/index.mjs

```js
import express from "express";
import dotenv from "dotenv";
import pool from "./db/pgPool.mjs";
import usersRouter from "./routes/users.js";

dotenv.config();
const app = express();
app.use(express.json());
app.use("/users", usersRouter);

// Error handler
app.use((err, req, res, next) => {
  console.error(err);
  res.status(500).json({ message: "Internal Server Error" });
});

const server = app.listen(process.env.PORT, () => {
  console.log("🚀 Server started at port", process.env.PORT);
});

// Graceful shutdown
process.on("SIGINT", async () => {
  console.log("🧹 Closing database pool...");
  await pool.end();
  server.close(() => process.exit(0));
});
```

---

### 🔹 Step 7: Transaction Example

```js
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

📘 **ব্যাখ্যা:**
`BEGIN`, `COMMIT`, `ROLLBACK` ব্যবহার করে একাধিক query কে একসাথে atomic operation হিসেবে চালানো হয়।

---

## 🧠 ৬. MongoDB ব্যবহার করা — `mongoose` সহ (Full Setup)

### 🔹 Step 1: Install

```bash
npm install express mongoose dotenv
```

### 🔹 Step 2: .env ফাইল

```
MONGO_URI=mongodb://localhost:27017/mydb
PORT=3000
```

### 🔹 Step 3: Connection Setup — src/db/mongoose.mjs

```js
import mongoose from "mongoose";
import dotenv from "dotenv";
dotenv.config();

export async function connectDB() {
  try {
    await mongoose.connect(process.env.MONGO_URI, {
      useNewUrlParser: true,
      useUnifiedTopology: true,
    });
    console.log("✅ MongoDB Connected");
  } catch (err) {
    console.error("MongoDB connection error:", err);
  }
}

export async function disconnectDB() {
  await mongoose.disconnect();
}
```

---

### 🔹 Step 4: Model তৈরি — src/models/User.mjs

```js
import mongoose from "mongoose";

const userSchema = new mongoose.Schema({
  name: { type: String, required: true },
  email: { type: String, required: true, unique: true },
}, { timestamps: true });

export default mongoose.model("User", userSchema);
```

---

### 🔹 Step 5: Service Layer — src/services/usersService.js

```js
import User from "../models/User.mjs";

export const getAllUsers = async () => await User.find().lean();
export const getUserById = async (id) => await User.findById(id).lean();
export const createUser = async (data) => await User.create(data);
```

---

### 🔹 Step 6: Route Layer — src/routes/users.js

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

router.post("/", async (req, res, next) => {
  try {
    const user = await usersService.createUser(req.body);
    res.status(201).json(user);
  } catch (err) { next(err); }
});

export default router;
```

---

### 🔹 Step 7: Main File — src/index.mjs

```js
import express from "express";
import dotenv from "dotenv";
import { connectDB, disconnectDB } from "./db/mongoose.mjs";
import usersRouter from "./routes/users.js";

dotenv.config();
const app = express();
app.use(express.json());
app.use("/users", usersRouter);

const server = app.listen(process.env.PORT, async () => {
  await connectDB();
  console.log(`🚀 Server running on port ${process.env.PORT}`);
});

process.on("SIGINT", async () => {
  await disconnectDB();
  server.close(() => process.exit(0));
});
```

---

## 💡 ৭. Database Logic কোথায় রাখবে?

* `services/` → data logic (query, CRUD)
* `controllers/` → request/response handle
* `routes/` → route mapping
* `db/` → connection setup

এভাবে কোড ক্লিন এবং maintainable থাকে।

---

## 🧩 ৮. Common Errors

| Error                   | কারণ                         | সমাধান                                    |
| ----------------------- | ---------------------------- | ----------------------------------------- |
| `ECONNREFUSED`          | DB সার্ভার চালু নেই          | সার্ভার চালাও (`pg_ctl start` / `mongod`) |
| `authentication failed` | ভুল username/password        | .env এ ঠিক করো                            |
| `E11000 duplicate key`  | Mongo unique field duplicate | `unique` ফিল্ড পরিবর্তন করো               |
| `too many clients`      | pool overflow                | pool size কমাও বা PgBouncer ব্যবহার করো   |

---

## 🔐 ৯. Best Practices (Production Tips)

1. .env ফাইলে credentials রাখো
2. SSL connection ব্যবহার করো
3. Connection pool এর max limit ঠিক রাখো
4. Proper indexing ব্যবহার করো
5. Transactions ব্যবহার করো critical operations এ
6. Graceful shutdown করো
7. Migration tool ব্যবহার করো (`knex`, `sequelize-cli`, etc.)
8. Database error logging যোগ করো

---

## 🧾 ১০. Summary

| ধাপ | কাজ                                      |
| --- | ---------------------------------------- |
| 1️⃣ | Library install করো (`pg` বা `mongoose`) |
| 2️⃣ | .env ফাইল সেট করো                        |
| 3️⃣ | Connection file তৈরি করো                 |
| 4️⃣ | Service layer এ query লিখো               |
| 5️⃣ | Route তৈরি করো                           |
| 6️⃣ | Error handling করো                       |
| 7️⃣ | Graceful shutdown ও test করো             |

---

## 🎓 ১১. Real-life Use Case উদাহরণ

**Use Case:**
একটি E-commerce সাইটে user register করলে সেটি `users` table-এ save হবে।

* Express route `/register` → POST request পাবে → DB তে insert হবে।
* Admin panel থেকে `/users` route এ GET করে সব user দেখা যাবে।

---