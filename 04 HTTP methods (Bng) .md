
---

# 🧩 এক বাক্যে সারসংক্ষেপ

**Express.js**-এ HTTP মেথডগুলো (যেমন: `GET`, `POST`, `PUT`, `PATCH`, `DELETE`, `HEAD`, `OPTIONS`) ব্যবহার করা হয় ক্লায়েন্টের উদ্দেশ্য বা **HTTP Action** বোঝাতে — এই গাইডে ধাপে ধাপে বোঝানো হয়েছে কিভাবে প্রতিটি মেথড কাজ করে, কখন ব্যবহার করতে হয়, সঠিকভাবে রাউট ও মিডলওয়্যার সাজাতে হয়, এবং রিয়েল-ওয়ার্ল্ড উদাহরণসহ সব কিছু।

---

## 🔹 ধাপ ১: Express অ্যাপ সেটআপ করা

প্রথমে প্রজেক্ট তৈরি করুন:

```bash
mkdir express-http-methods-detailed
cd express-http-methods-detailed
npm init -y
npm i express
```

তারপর `app.js` তৈরি করুন:

```js
const express = require('express');
const app = express();
const port = 3000;

app.use(express.json());
app.use(express.urlencoded({ extended: true }));

app.get('/', (req, res) => res.send('Hello Express'));
app.listen(port, () => console.log(`Server running at http://localhost:${port}`));
```

চালাতে:

```bash
node app.js
```

---

## 🔹 ধাপ ২: HTTP মেথড — কী ও কখন ব্যবহার করবেন

| মেথড        | কাজ                              | কখন ব্যবহার করবেন            | বৈশিষ্ট্য                 |
| ----------- | -------------------------------- | ---------------------------- | ------------------------- |
| **GET**     | ডেটা **পড়া বা ফেচ** করা         | লিস্ট, ডিটেইলস, সার্চ        | নিরাপদ (Safe), Idempotent |
| **HEAD**    | GET-এর মতো, কিন্তু **বডি ছাড়া** | হেলথ চেক, হেডার দেখা         | Safe, Idempotent          |
| **POST**    | নতুন **ডেটা তৈরি করা**           | ফর্ম সাবমিট, ইউজার রেজিস্টার | Not idempotent            |
| **PUT**     | **পুরো রিসোর্স রিপ্লেস** করা     | আপডেটের সময় সব ফিল্ড        | Idempotent                |
| **PATCH**   | **আংশিক আপডেট** করা              | কিছু ফিল্ড পরিবর্তন          | Partial update            |
| **DELETE**  | ডেটা মুছে ফেলা                   | আইটেম রিমুভ                  | Idempotent                |
| **OPTIONS** | সার্ভার কী কী মেথড সাপোর্ট করে   | CORS প্রিফ্লাইট চেক          | Safe                      |

🟢 **মূলনীতি:** কখনোই `GET` দিয়ে ডেটা পরিবর্তন করবেন না।

---

## 🔹 ধাপ ৩: রাউট ডিফাইন করা

### 👉 সাধারণ রাউট

```js
app.get('/items', (req, res) => { /* লিস্ট দেখানো */ });
app.post('/items', (req, res) => { /* নতুন আইটেম যোগ */ });
app.get('/items/:id', (req, res) => { /* নির্দিষ্ট আইটেম */ });
```

### 👉 Route chaining (একসাথে সব CRUD)

```js
app.route('/items/:id')
  .get(getItem)
  .put(replaceItem)
  .patch(updateItem)
  .delete(deleteItem);
```

### 👉 Modular Router

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

## 🔹 ধাপ ৪: Request Data Access করা

| ধরন           | উৎস          | উদাহরণ                     |
| ------------- | ------------ | -------------------------- |
| `req.params`  | URL params   | `/users/:id`               |
| `req.query`   | Query string | `/search?q=test&page=2`    |
| `req.body`    | Request body | `POST` বা `PUT` রিকোয়েস্ট |
| `req.headers` | Headers      | `req.get('User-Agent')`    |

📦 **File Upload (multer):**

```js
const multer = require('multer');
const upload = multer({ dest: 'uploads/' });

app.post('/profile', upload.single('avatar'), (req, res) => {
  res.json({ file: req.file, body: req.body });
});
```

---

## 🔹 ধাপ ৫: CRUD উদাহরণ (in-memory)

```js
const express = require('express');
const app = express();
app.use(express.json());

let users = [];
let nextId = 1;

// GET (লিস্ট)
app.get('/users', (req, res) => {
  const { name } = req.query;
  const results = name ? users.filter(u => u.name.includes(name)) : users;
  res.json(results);
});

// POST (তৈরি)
app.post('/users', (req, res) => {
  const { name, age } = req.body;
  if (!name) return res.status(400).json({ error: 'Name required' });
  const user = { id: nextId++, name, age };
  users.push(user);
  res.status(201).json(user);
});

// GET (একজন)
app.get('/users/:id', (req, res) => {
  const id = Number(req.params.id);
  const user = users.find(u => u.id === id);
  if (!user) return res.status(404).json({ error: 'Not found' });
  res.json(user);
});

// PUT (পুরো আপডেট)
app.put('/users/:id', (req, res) => {
  const id = Number(req.params.id);
  const idx = users.findIndex(u => u.id === id);
  if (idx === -1) return res.status(404).json({ error: 'Not found' });
  const { name, age } = req.body;
  users[idx] = { id, name, age };
  res.json(users[idx]);
});

// PATCH (আংশিক আপডেট)
app.patch('/users/:id', (req, res) => {
  const id = Number(req.params.id);
  const user = users.find(u => u.id === id);
  if (!user) return res.status(404).json({ error: 'Not found' });
  Object.assign(user, req.body);
  res.json(user);
});

// DELETE (মুছে ফেলা)
app.delete('/users/:id', (req, res) => {
  const id = Number(req.params.id);
  users = users.filter(u => u.id !== id);
  res.sendStatus(204);
});

app.listen(3000);
```

---

## 🔹 ধাপ ৬: `curl` দিয়ে টেস্ট

```bash
# তৈরি
curl -X POST -H "Content-Type: application/json" -d '{"name":"Alice"}' http://localhost:3000/users

# লিস্ট দেখা
curl http://localhost:3000/users

# নির্দিষ্ট ইউজার
curl http://localhost:3000/users/1

# আপডেট
curl -X PUT -H "Content-Type: application/json" -d '{"name":"Alicia","age":30}' http://localhost:3000/users/1

# আংশিক আপডেট
curl -X PATCH -H "Content-Type: application/json" -d '{"age":31}' http://localhost:3000/users/1

# মুছে ফেলা
curl -X DELETE http://localhost:3000/users/1
```

---

## 🔹 ধাপ ৭: স্ট্যাটাস কোড ব্যাখ্যা

| কোড | মানে                  | ব্যবহার     |
| --- | --------------------- | ----------- |
| 200 | OK                    | সফল GET     |
| 201 | Created               | নতুন তৈরি   |
| 204 | No Content            | সফল DELETE  |
| 400 | Bad Request           | ভুল ইনপুট   |
| 404 | Not Found             | রিসোর্স নেই |
| 500 | Internal Server Error | সার্ভার এরর |

---

## 🔹 ধাপ ৮: Error Handling

```js
app.use((err, req, res, next) => {
  console.error(err);
  res.status(err.status || 500).json({ error: err.message });
});
```

---

## 🔹 ধাপ ৯: Middleware Order

1. Security (helmet)
2. Logger (morgan)
3. Body parser
4. CORS
5. Auth
6. Routes
7. Error handler

---

## 🔹 ধাপ ১০: ইনপুট ভ্যালিডেশন

`express-validator` ব্যবহার করুন:

```js
const { body, validationResult } = require('express-validator');

app.post('/users', [
  body('name').isString().notEmpty(),
  body('age').optional().isInt()
], (req, res) => {
  const errors = validationResult(req);
  if (!errors.isEmpty()) return res.status(400).json({ errors: errors.array() });
});
```

---

## 🔹 ধাপ ১১: CORS ও OPTIONS

```js
const cors = require('cors');
app.use(cors({ origin: 'https://example.com' }));
```

---

## 🔹 ধাপ ১২: ক্যাশিং ও ETag

```js
res.set('Cache-Control', 'public, max-age=3600');
```

Express স্বয়ংক্রিয়ভাবে ETag হ্যান্ডেল করে।

---

## 🔹 ধাপ ১৩: Idempotency ধারণা

* **Safe:** GET, HEAD, OPTIONS
* **Idempotent:** GET, PUT, DELETE, PATCH
* POST সাধারণত Idempotent নয় (একই রিকোয়েস্টে বারবার নতুন ডেটা তৈরি হতে পারে)।

---

## 🔹 ধাপ ১৪: ফাইল আপলোড ও স্ট্রিমিং

```js
const fs = require('fs');
app.get('/download/:file', (req, res) => {
  const stream = fs.createReadStream(`./data/${req.params.file}`);
  stream.pipe(res);
});
```

---

## 🔹 ধাপ ১৫: সিকিউরিটি টিপস

✅ HTTPS ব্যবহার করুন
✅ Helmet, CORS, Rate Limit ব্যবহার করুন
✅ ইনপুট ভ্যালিডেশন করুন
✅ JWT/Auth ব্যবহার করুন
✅ GET দিয়ে কিছু পরিবর্তন করবেন না

---

## 🔹 ধাপ ১৬: টেস্টিং

```js
const request = require('supertest');
const app = require('./app');

test('create user', async () => {
  const res = await request(app)
    .post('/users')
    .send({ name: 'Alice' })
    .expect(201);
  expect(res.body).toHaveProperty('id');
});
```

---

## 🔹 ধাপ ১৭: RESTful টিপস

✅ রিসোর্স নাম **noun** আকারে দিন
✅ `/orders`, `/users`, `/products`
✅ Nested রুট কম ব্যবহার করুন
✅ API version দিন (`/v1/users`)

---

## 🔹 ধাপ ১৮: সাধারণ ভুল

* `req.body` undefined → `express.json()` বাদ পড়েছে
* CORS error → `cors()` ভুলভাবে কনফিগার
* Async error → `try/catch` বাদ
* রুট অর্ডার ভুল

---

## 🔹 ধাপ ১৯: Advanced PATCH ও Conditional Requests

`If-Match` + `ETag` ব্যবহার করে ডেটা কনফ্লিক্ট এড়াতে পারেন (optimistic concurrency control)।

---

## 🔹 ধাপ ২০: ফোল্ডার স্ট্রাকচার

```
/project
  /src
    /routes
    /controllers
    /services
    /middlewares
    app.js
```

---

## 🔹 ধাপ ২১: দরকারি Middleware

* `helmet` — সিকিউরিটি হেডার
* `cors` — CORS হ্যান্ডেল
* `morgan` — লগিং
* `multer` — ফাইল আপলোড
* `express-validator` / `joi` — ইনপুট ভ্যালিডেশন
* `supertest` — টেস্টিং

---

## 🔹 ধাপ ২২: প্রোডাকশনে যাবার আগে চেকলিস্ট

✅ HTTPS
✅ Auth
✅ Validation
✅ CORS policy
✅ Rate limit
✅ Error handler
✅ Logger
✅ Monitoring

---

## 🔹 ধাপ ২৩: দ্রুত সিদ্ধান্ত গাইড

| কাজ         | মেথড    |
| ----------- | ------- |
| ডেটা ফেচ    | GET     |
| তৈরি        | POST    |
| রিপ্লেস     | PUT     |
| আংশিক আপডেট | PATCH   |
| মুছে ফেলা   | DELETE  |
| প্রিফ্লাইট  | OPTIONS |

---

