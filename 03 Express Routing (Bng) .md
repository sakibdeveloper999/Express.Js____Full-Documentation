এখন আমরা Express.js-এর **Routing System** সম্পূর্ণভাবে বাংলায় ব্যাখ্যা করব — একদম শুরু থেকে বাস্তব প্রজেক্ট স্টাইলে।
তুমি শেষে বুঝবে, কীভাবে Express.js রাউটিং কাজ করে, কেন লাগে, এবং কীভাবে বড় প্রজেক্টে সঠিকভাবে ব্যবহার করতে হয়।

---

# 🌐 **Express.js Routing — পূর্ণাঙ্গ বাংলা টিউটোরিয়াল**

---

## 🧠 ১. Express.js Routing কী?

Routing মানে হলো, **একটি নির্দিষ্ট URL এবং HTTP Method অনুযায়ী** সার্ভারের কোন কোড চলবে, সেটা নির্ধারণ করা।
অর্থাৎ, কেউ যদি ব্রাউজারে `/home` বা `/users` বা `/api/products` এ যায়, তাহলে Express জানে — “এই রিকোয়েস্টের জন্য কোন ফাংশন রান হবে”।

🔹 সহজভাবে:

```
User Request (GET /users) → Express.js Route → Response (user list)
```

---

## ⚙️ ২. প্রথমে একটা Express App তৈরি করি

👉 ধাপে ধাপে শুরু করি:

### Step 1: প্রজেক্ট সেটআপ

```bash
mkdir express-routing-demo
cd express-routing-demo
npm init -y
npm install express
```

### Step 2: `index.js` ফাইল তৈরি করো

```js
// index.js
import express from "express"; // যদি ES Modules হয়
// const express = require("express"); // CommonJS হলে

const app = express();
const PORT = 3000;

// Home route
app.get("/", (req, res) => {
  res.send("Express Routing Tutorial এ স্বাগতম!");
});

app.listen(PORT, () => {
  console.log(`Server running at http://localhost:${PORT}`);
});
```

👉 এবার চালাও:

```bash
node index.js
```

এবং ব্রাউজারে যাও:
🔗 `http://localhost:3000`
তুমি দেখবে: **“Express Routing Tutorial এ স্বাগতম!”**

---

## 🛣️ ৩. Express Routes কিভাবে কাজ করে?

Express রাউটিং মানে হচ্ছে:

```js
app.METHOD(PATH, HANDLER)
```

যেখানে:

* **METHOD:** HTTP Method (যেমন GET, POST, PUT, DELETE)
* **PATH:** কোন URL-এ রাউট কাজ করবে
* **HANDLER:** একটা ফাংশন যা Request পেলে Response পাঠায়

### উদাহরণ:

```js
app.get("/about", (req, res) => {
  res.send("About Page");
});

app.post("/contact", (req, res) => {
  res.send("Contact Form Submitted!");
});
```

---

## 🧩 ৪. HTTP Method অনুযায়ী রাউট

| Method   | কাজ                       |
| -------- | ------------------------- |
| `GET`    | ডাটা পড়া                  |
| `POST`   | নতুন ডাটা তৈরি            |
| `PUT`    | পুরনো ডাটা সম্পূর্ণ আপডেট |
| `PATCH`  | আংশিক আপডেট               |
| `DELETE` | ডাটা মুছে ফেলা            |

### উদাহরণ:

```js
app.get("/users", (req, res) => res.send("সব ইউজার দেখাও"));
app.post("/users", (req, res) => res.send("নতুন ইউজার তৈরি কর"));
app.put("/users/:id", (req, res) => res.send("পুরনো ইউজার আপডেট কর"));
app.delete("/users/:id", (req, res) => res.send("ইউজার ডিলিট কর"));
```

---

## 🧮 ৫. Route Parameter এবং Query Parameter

### 🟢 Route Parameter

```js
app.get("/users/:id", (req, res) => {
  res.send(`তুমি ইউজার আইডি ${req.params.id} দেখতে চাও`);
});
```

➡️ এখানে `/users/5` মানে `req.params.id = 5`

### 🟣 Query Parameter

```js
app.get("/search", (req, res) => {
  const query = req.query.q;
  res.send(`তুমি "${query}" খুঁজছ`);
});
```

➡️ `/search?q=node` হলে `req.query.q = "node"`

---

## 🧰 ৬. Middleware (রাউটের আগে বা মাঝে কিছু কাজ করা)

Middleware হলো এমন ফাংশন যা **request আসার পর response পাঠানোর আগে** কিছু কাজ করে।

```js
app.use(express.json()); // body-parser হিসেবে কাজ করে

app.use((req, res, next) => {
  console.log(`Request আসছে: ${req.method} ${req.path}`);
  next(); // পরের রাউটে পাঠাও
});
```

### Route-level middleware:

```js
function checkAuth(req, res, next) {
  if (!req.headers.authorization) return res.status(401).send("Unauthorized!");
  next();
}

app.get("/secret", checkAuth, (req, res) => {
  res.send("তুমি সফলভাবে অনুমোদিত!");
});
```

---

## 🧱 ৭. Express Router — Modular Routing

যখন প্রজেক্ট বড় হয়, তখন সব রাউট `index.js`-এ রাখলে কোড গুলিয়ে যায়।
তাই আলাদা ফাইল তৈরি করে রাউটগুলো ভাগ করে নিতে হয়।

### 👉 `routes/users.js`

```js
import express from "express";
const router = express.Router();

let users = [{ id: 1, name: "Sakib" }];

// সব ইউজার দেখাও
router.get("/", (req, res) => res.json(users));

// নির্দিষ্ট ইউজার দেখাও
router.get("/:id", (req, res) => {
  const user = users.find(u => u.id == req.params.id);
  if (!user) return res.status(404).send("User not found!");
  res.json(user);
});

// নতুন ইউজার যোগ করো
router.post("/", (req, res) => {
  const newUser = { id: Date.now(), name: req.body.name };
  users.push(newUser);
  res.status(201).json(newUser);
});

export default router;
```

### 👉 `index.js`

```js
import express from "express";
import usersRouter from "./routes/users.js";

const app = express();
app.use(express.json());

// সব ইউজার রাউট "/users" এর অধীনে থাকবে
app.use("/users", usersRouter);

app.listen(3000, () => console.log("Server is running..."));
```

---

## 🧑‍💻 ৮. Route Order — খুবই গুরুত্বপূর্ণ!

Express উপরের দিক থেকে রাউট চেক করে।

```js
app.get("/users/me", (req, res) => res.send("This is you"));
app.get("/users/:id", (req, res) => res.send("Another user"));
```

যদি `/users/:id` আগে রাখো, তাহলে `/users/me` কখনো রান হবে না — কারণ `:id` “me”-কেও ধরবে।

---

## 🧠 ৯. Error Handling Middleware

যখন কোন রাউটে ভুল হয়, তখন error handler middleware সব error ধরবে।

```js
app.use((err, req, res, next) => {
  console.error("Error:", err.message);
  res.status(500).json({ error: "Server error occurred!" });
});
```

👉 এই middleware সবশেষে রাখতে হয়।

---

## 🧱 ১০. Static File Routing

Express দিয়ে HTML, CSS, ছবি ইত্যাদি ফাইলও serve করা যায়।

```js
app.use(express.static("public"));
```

যদি তোমার `public/index.html` থাকে, তাহলে `/` রাউটে সেটাই দেখাবে।

---

## 🧩 ১১. Route Grouping এবং Chaining

একই path-এর জন্য একাধিক Method ব্যবহার করতে চাইলে:

```js
app.route("/books")
  .get((req, res) => res.send("সব বই দেখাও"))
  .post((req, res) => res.send("নতুন বই যোগ কর"));
```

---

## 🧑‍🏫 ১২. ছোট একটা CRUD API প্রজেক্ট (Users API)

```js
// routes/users.js
import express from "express";
const router = express.Router();

let users = [];

router.get("/", (req, res) => res.json(users));

router.post("/", (req, res) => {
  const user = { id: Date.now(), name: req.body.name };
  users.push(user);
  res.status(201).json(user);
});

router.get("/:id", (req, res) => {
  const user = users.find(u => u.id == req.params.id);
  if (!user) return res.status(404).send("User not found");
  res.json(user);
});

router.put("/:id", (req, res) => {
  const user = users.find(u => u.id == req.params.id);
  if (!user) return res.status(404).send("User not found");
  user.name = req.body.name;
  res.json(user);
});

router.delete("/:id", (req, res) => {
  users = users.filter(u => u.id != req.params.id);
  res.status(204).end();
});

export default router;
```

```js
// index.js
import express from "express";
import usersRouter from "./routes/users.js";

const app = express();
app.use(express.json());
app.use("/users", usersRouter);

app.listen(3000, () => console.log("✅ Server running at port 3000"));
```

---

## 🧾 ১৩. রাউট টেস্ট করার কিছু পদ্ধতি

1. **Browser বা Postman দিয়ে GET রিকোয়েস্ট পাঠাও**
2. **POST/PUT/DELETE রিকোয়েস্ট** → Postman বা Curl ব্যবহার করো

   ```bash
   curl -X POST http://localhost:3000/users -H "Content-Type: application/json" -d '{"name":"Sakib"}'
   ```

---

## 🧭 ১৪. ভালো রাউটিংয়ের Best Practices

✅ ছোট ছোট ফাইলে ভাগ করো (`routes/`, `controllers/`, `middlewares/`)
✅ Route নাম স্পষ্ট রাখো (`/api/users`, `/api/products`)
✅ Error Handling আলাদা করো
✅ Validation যোগ করো (যেমন express-validator)
✅ রাউটগুলো RESTful প্যাটার্নে তৈরি করো

---

## 🧩 ১৫. প্রজেক্ট স্ট্রাকচার সাজেশন

```
express-routing-demo/
│
├── index.js
├── routes/
│   └── users.js
├── controllers/
│   └── usersController.js
├── middlewares/
│   └── auth.js
└── package.json
```

---

## 🎯 সারসংক্ষেপ

| ধাপ | কী হয়                                 |
| --- | ------------------------------------- |
| 1️⃣ | Express App তৈরি                      |
| 2️⃣ | Route define (app.get, app.post...)   |
| 3️⃣ | Middleware ব্যবহার                    |
| 4️⃣ | express.Router() দিয়ে রাউট আলাদা করা  |
| 5️⃣ | Error Handler যোগ করা                 |
| 6️⃣ | Static Files Serve করা                |
| 7️⃣ | প্রজেক্ট স্ট্রাকচার সুন্দরভাবে সাজানো |

---

## 🔚 শেষ কথা

Express.js Routing আসলে তোমার **backend অ্যাপের backbone**।
প্রতিটি API endpoint মানে একটা রাউট — তাই এটা ভালোভাবে বুঝতে পারলে, তুমি খুব সহজেই বড় REST API বা MERN প্রজেক্ট বানাতে পারবে।

---