
---

## 🧠 1️⃣ Middleware কী (সহজ সংজ্ঞা)

**Express.js**-এ
**Middleware** মানে হলো —
➡️ “এমন কিছু ফাংশন যা ক্লায়েন্টের রিকোয়েস্ট (Request) এবং সার্ভারের রেসপন্স (Response)-এর মাঝে কাজ করে।”

এই ফাংশনগুলো পারে:

* `req` (request object) এবং `res` (response object) পরিবর্তন বা অ্যাক্সেস করতে।
* যেকোনো কোড চালাতে (যেমন লগ করা, যাচাই করা, অথেনটিকেশন ইত্যাদি)।
* সরাসরি রেসপন্স পাঠিয়ে রিকোয়েস্ট–রেসপন্স সাইকেল শেষ করতে।
* অথবা `next()` কল করে পরবর্তী middleware-এ যেতে।

---

## ⚙️ 2️⃣ Middleware কিভাবে কাজ করে (ভিতরের লজিক)

যখন কোনো ক্লায়েন্ট রিকোয়েস্ট পাঠায় → Express এটা এমনভাবে প্রসেস করে 👇

```
Client → [Middleware 1] → [Middleware 2] → [Middleware 3] → Route Handler → Response
```

প্রতিটি Middleware পায় তিনটি প্যারামিটার:

```js
(req, res, next)
```

* **req:** ক্লায়েন্ট থেকে আসা ডেটা (headers, params, body ইত্যাদি)
* **res:** সার্ভারের রেসপন্স পাঠানোর অবজেক্ট
* **next:** পরবর্তী middleware বা রাউট হ্যান্ডলারে যাওয়ার জন্য ফাংশন

👉 যদি তুমি `next()` না লেখো, তাহলে এখানেই রিকোয়েস্ট থেমে যাবে।

---

## 🧩 3️⃣ Middleware ফাংশনের মৌলিক কাঠামো

```js
function middlewareName(req, res, next) {
  console.log('Middleware চলছে...');
  next(); // পরের middleware এ যাবে
}
```

**app-এ ব্যবহার (রেজিস্টার) করা:**

```js
const express = require('express');
const app = express();

app.use(middlewareName); // গ্লোবালি প্রয়োগ করা
```

---

## 🧱 4️⃣ Express.js এ Middleware-এর ধরন

Express-এ সাধারণত ৫ ধরনের Middleware থাকে 👇

| ধরন                   | কাজ                                | উদাহরণ                  |
| --------------------- | ---------------------------------- | ----------------------- |
| **Application-level** | পুরো অ্যাপে বা নির্দিষ্ট রাউটে চলে | `app.use()`             |
| **Router-level**      | নির্দিষ্ট রাউটার ফাইলের জন্য       | `router.use()`          |
| **Built-in**          | Express এর সাথে বিল্ট-ইন আসে       | `express.json()`        |
| **Third-party**       | npm থেকে ইনস্টল করতে হয়            | `morgan`, `cors`        |
| **Error-handling**    | এরর ধরতে ও হ্যান্ডেল করতে          | `(err, req, res, next)` |

---

## 🧭 5️⃣ Application-Level Middleware (গ্লোবাল মিডলওয়্যার)

```js
const express = require('express');
const app = express();

// কাস্টম লগার middleware
function logger(req, res, next) {
  console.log(`${req.method} ${req.url}`);
  next();
}

app.use(logger); // সব রাউটে প্রয়োগ হবে

app.get('/', (req, res) => {
  res.send('Home Page');
});

app.get('/about', (req, res) => {
  res.send('About Page');
});

app.listen(3000);
```

🧠 কনসোলে দেখা যাবে:

```
GET /
GET /about
```

---

## 🧭 6️⃣ Router-Level Middleware

এই Middleware শুধু নির্দিষ্ট রাউটার ফাইলের মধ্যে কাজ করে।

```js
// routes/admin.js
const express = require('express');
const router = express.Router();

// শুধু admin রাউটের জন্য middleware
function checkAdmin(req, res, next) {
  const isAdmin = true; // ধরা যাক চেক করা হচ্ছে
  if (!isAdmin) return res.status(403).send('Forbidden');
  next();
}

router.use(checkAdmin);

router.get('/dashboard', (req, res) => {
  res.send('Welcome Admin!');
});

module.exports = router;

// main app.js
const app = express();
const adminRoutes = require('./routes/admin');
app.use('/admin', adminRoutes);
```

---

## ⚙️ 7️⃣ Built-in Middleware (Express এর বিল্ট-ইন ফাংশন)

| Middleware                 | কাজ                                    |
| -------------------------- | -------------------------------------- |
| `express.json()`           | JSON ডেটা পার্স করে `req.body` তে রাখে |
| `express.urlencoded()`     | Form data পার্স করে                    |
| `express.static('folder')` | Static ফাইল সার্ভ করে (CSS, JS, ইমেজ)  |

```js
app.use(express.json());
app.use(express.static('public'));
app.use(express.urlencoded({ extended: true }));
```

---

## 🧰 8️⃣ Third-Party Middleware (npm থেকে ইন্সটল করা)

| Middleware        | কাজ               |
| ----------------- | ----------------- |
| `morgan`          | রিকোয়েস্ট লগ রাখে |
| `cors`            | CORS সক্রিয় করে   |
| `cookie-parser`   | কুকি পার্স করে    |
| `express-session` | সেশন ম্যানেজ করে  |
| `helmet`          | নিরাপত্তা বাড়ায়   |

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

## ⚠️ 9️⃣ Error-Handling Middleware

এরর ধরতে এবং রেসপন্স পাঠাতে ব্যবহৃত হয়।

> এই Middleware-এ অবশ্যই ৪টা প্যারামিটার থাকতে হবে: `(err, req, res, next)`

```js
function errorHandler(err, req, res, next) {
  console.error(err.message);
  res.status(500).json({ error: 'Something went wrong!' });
}

app.use(errorHandler);
```

যদি কোনো রাউটে এরর হয়:

```js
app.get('/crash', (req, res, next) => {
  try {
    throw new Error('Server broke!');
  } catch (err) {
    next(err); // errorHandler এ যাবে
  }
});
```

---

## 🔁 🔟 Middleware এর কাজের ধারা (Order গুরুত্বপূর্ণ)

Express একে একে Middleware চালায় — যেভাবে তুমি রেজিস্টার করো।

```js
app.use(first);
app.use(second);
app.get('/', (req, res) => res.send('done!'));
```

➡️ Execution order:

```
first → second → route handler
```

👉 যদি `first` এ `next()` না ডাকা হয়, তাহলে বাকিগুলো চলবে না।

---

## 🧠 11️⃣ বাস্তব জীবনের Middleware ব্যবহার

| ব্যবহার             | উদ্দেশ্য                                 |
| ------------------- | ---------------------------------------- |
| **Logging**         | কে কোন রিকোয়েস্ট পাঠিয়েছে তা রেকর্ড রাখা |
| **Authentication**  | লগইন বা টোকেন যাচাই করা                  |
| **Data Validation** | ডেটা সঠিক কি না যাচাই করা                |
| **Security**        | নিরাপত্তা হেডার যুক্ত করা                |
| **Compression**     | রেসপন্স সাইজ ছোট করা                     |
| **Error Tracking**  | এরর ক্যাচ করে রিপোর্ট করা                |

---

## 💡 12️⃣ একাধিক Middleware একসাথে ব্যবহার

```js
function auth(req, res, next) {
  if (!req.headers.token) return res.status(401).send('Unauthorized');
  next();
}

function logRequest(req, res, next) {
  console.log(`${req.method} - ${req.url}`);
  next();
}

app.use(logRequest); // সব রাউটে
app.get('/private', auth, (req, res) => res.send('Welcome authorized user!'));
```

🔸 এখানে:

* `logRequest` সব রাউটে চলবে
* `auth` শুধু `/private` রাউটে চলবে

---

## ⚡ 13️⃣ Async Middleware Example (Promise / async-await সহ)

```js
app.use(async (req, res, next) => {
  try {
    const user = await getUserFromDB();
    req.user = user;
    next();
  } catch (err) {
    next(err); // Error Handler এ যাবে
  }
});
```

---

## 🧾 14️⃣ Middleware লেখার কিছু ভালো অভ্যাস (Best Practices)

✅ সবসময় `next()` কল করো, যদি রেসপন্স না পাঠাও।
✅ প্রতিটি Middleware-কে একটাই কাজের জন্য ব্যবহার করো (Single Responsibility)।
✅ Middleware-এর ক্রম ঠিক রাখো (Order matters)।
✅ এরর Middleware সবশেষে রাখো।
✅ Event loop ব্লক কোরো না, async I/O ব্যবহার করো।
✅ পুনর্ব্যবহারযোগ্য ও পরিষ্কার কোড লিখো।

---

## 🔍 15️⃣ সারাংশ (Quick Revision)

| বিষয়          | সারসংক্ষেপ                                   |
| ------------- | -------------------------------------------- |
| Middleware    | Request ও Response-এর মাঝে কাজ করে এমন ফাংশন |
| Arguments     | `(req, res, next)`                           |
| Built-in      | `express.json()`, `express.static()`         |
| Custom        | `app.use(myFunction)`                        |
| Error Handler | `(err, req, res, next)`                      |
| Order         | উপরের দিক থেকে নিচের দিকে                    |
| Use Cases     | Logging, Auth, Validation, Security ইত্যাদি  |

---
