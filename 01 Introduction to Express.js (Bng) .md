
---

# 🚀 **Express.js পরিচিতি **

---

## 🧩 **১. Express.js কী?**

**Express.js** হলো একটি **Node.js ভিত্তিক ব্যাকএন্ড ওয়েব অ্যাপ্লিকেশন ফ্রেমওয়ার্ক**।
এর কাজ হলো — ওয়েব সার্ভার তৈরি করা, API বানানো, রাউট হ্যান্ডল করা, এবং রিকোয়েস্ট-রেসপন্স ম্যানেজ করা সহজ করে তোলা।

সহজভাবে বললে 👇

> **Node.js** হচ্ছে ইঞ্জিন (server runtime), আর **Express.js** হচ্ছে সেই ইঞ্জিনের গাড়ির বডি — মানে সার্ভার তৈরি করার জন্য প্রস্তুত কাঠামো।

Express.js-কে প্রায়ই বলা হয়:

> “Node.js এর সবচেয়ে জনপ্রিয় ও স্ট্যান্ডার্ড ওয়েব ফ্রেমওয়ার্ক।”

---

## ⚙️ **২. কেন Express.js ব্যবহার করবো?**

Node.js দিয়ে সরাসরি সার্ভার বানানো কষ্টকর।
প্রতিটা রিকোয়েস্ট আলাদা করে হ্যান্ডল করা, ডেটা পার্স করা, রাউট বানানো—সব কিছু অনেক কোড লাগে।

Express.js এই সব কাজ সহজ করে দেয়।
এটা দেয়:

* ✅ রাউটিং সিস্টেম
* ✅ মিডলওয়্যার সাপোর্ট
* ✅ এরর হ্যান্ডলিং
* ✅ RESTful API তৈরি করার সহজ উপায়
* ✅ টেমপ্লেট ইঞ্জিন সাপোর্ট

ফলাফল:

> কম কোডে বেশি কাজ, এবং দ্রুত সার্ভার তৈরি।

---

## 🏗️ **৩. Express.js এর মূল ফিচারগুলো**

| ফিচার                   | ব্যাখ্যা                                                                                    |
| ----------------------- | ------------------------------------------------------------------------------------------- |
| 🧭 **Routing**          | বিভিন্ন URL এবং HTTP মেথড (GET, POST, PUT, DELETE) এর জন্য আলাদা রাউট তৈরি করা যায়।        |
| ⚙️ **Middleware**       | প্রতিটি রিকোয়েস্টের আগে বা পরে কিছু কাজ করানো যায় — যেমন authentication, logging ইত্যাদি। |
| 📦 **Static Files**     | HTML, CSS, JS, image ফাইল সার্ভ করা যায়।                                                   |
| 💾 **Template Engines** | EJS, Pug, Handlebars ইত্যাদি দিয়ে ডাইনামিক ওয়েব পেজ বানানো যায়।                          |
| 🌐 **REST API Support** | JSON ফরম্যাটে API তৈরি করা সহজ।                                                             |
| ⚡ **Performance**       | Node.js এর asynchronous nature থাকার কারণে দ্রুত কাজ করে।                                   |

---

## 🧠 **৪. Express.js কিভাবে কাজ করে**

একটি ক্লায়েন্ট (যেমন ব্রাউজার বা ফ্রন্টএন্ড অ্যাপ) যখন সার্ভারে রিকোয়েস্ট পাঠায়, তখন নিম্নলিখিত ধাপগুলো ঘটে 👇

1. ক্লায়েন্ট থেকে রিকোয়েস্ট আসে
2. মিডলওয়্যার রিকোয়েস্ট প্রসেস করে (যেমন authentication, validation)
3. রাউট হ্যান্ডলার নির্ধারণ করে কী রেসপন্স যাবে
4. সার্ভার ক্লায়েন্টকে রেসপন্স পাঠায়

**ফ্লো ডায়াগ্রাম:**

```
[ Client ] → [ Express Server ]
     ↓
 Middleware (auth, body-parser, etc.)
     ↓
 Route Handler (GET / POST / PUT)
     ↓
[ Response sent to client ]
```

---

## 🧰 **৫. “Hello World” উদাহরণ**

```js
// ১️⃣ Express ইম্পোর্ট করা
const express = require('express');

// ২️⃣ অ্যাপ তৈরি করা
const app = express();

// ৩️⃣ একটি রাউট তৈরি করা
app.get('/', (req, res) => {
  res.send('হ্যালো! এটি Express.js সার্ভার।');
});

// ৪️⃣ সার্ভার চালানো
app.listen(3000, () => {
  console.log('সার্ভার চলছে http://localhost:3000');
});
```

👉 টার্মিনালে চালাও:

```bash
node app.js
```

তারপর ব্রাউজারে দেখো:
**[http://localhost:3000](http://localhost:3000)** → “হ্যালো! এটি Express.js সার্ভার।”

---

## 🔁 **৬. Middleware কীভাবে কাজ করে**

**Middleware** হলো এমন ফাংশন যা প্রতিটি রিকোয়েস্টের মাঝে চলে।
এগুলো ব্যবহার করা যায়:

* Authentication
* Logging
* Error handling
* JSON parsing

উদাহরণ:

```js
app.use((req, res, next) => {
  console.log('রিকোয়েস্ট এসেছে:', req.method, req.url);
  next(); // পরের middleware বা রাউটে যাবে
});
```

---

## 🌐 **৭. Routing (রাউটিং) উদাহরণ**

রাউট মানে হলো, কোন URL এ গেলে কী রেসপন্স যাবে তা নির্ধারণ করা।

```js
app.get('/', (req, res) => res.send('Home Page'));
app.get('/about', (req, res) => res.send('About Page'));
app.post('/login', (req, res) => res.send('Login Successful'));
```

এভাবে খুব সহজে তোমার সার্ভারে একাধিক পথ (route) বানানো যায়।

---

## 📦 **৮. Static Files সার্ভ করা**

`express.static()` ব্যবহার করে ফাইল সার্ভ করা যায়:

```js
app.use(express.static('public'));
```

এখন `public` ফোল্ডারে থাকা HTML, CSS, বা JS ফাইল সরাসরি ব্রাউজারে দেখা যাবে।

---

## 💡 **৯. JSON এবং API কাজ করা**

Express দিয়ে RESTful API বানানো অনেক সহজ।

```js
app.use(express.json());

app.post('/api/data', (req, res) => {
  console.log(req.body);
  res.json({ message: 'ডেটা সফলভাবে পাওয়া গেছে' });
});
```

এইভাবে Express.js দিয়ে মোবাইল বা ফ্রন্টএন্ড অ্যাপের জন্য ব্যাকএন্ড API তৈরি করা যায়।

---

## ⚙️ **১০. Template Engine সাপোর্ট**

ডাইনামিক HTML পেজ তৈরি করতে Express বিভিন্ন টেমপ্লেট ইঞ্জিন সাপোর্ট করে (যেমন EJS, Pug)।

```js
app.set('view engine', 'ejs');
app.get('/profile', (req, res) => {
  res.render('profile', { name: 'সাকিব', age: 21 });
});
```

---

## 🧩 **১১. ফোল্ডার স্ট্রাকচার (উদাহরণ)**

```
my-express-app/
│
├── app.js
├── package.json
├── routes/
│   ├── userRoutes.js
│   └── productRoutes.js
├── public/
│   ├── style.css
│   └── script.js
├── views/
│   └── index.ejs
└── middleware/
    └── auth.js
```

এই স্ট্রাকচারে প্রোজেক্ট অনেক বেশি পরিষ্কার ও বড় অ্যাপ বানানো সহজ হয়।

---

## 🧾 **১২. Express.js এর ব্যবহার**

✅ React / Angular / Vue ফ্রন্টএন্ডের ব্যাকএন্ড হিসেবে
✅ RESTful API তৈরি
✅ Authentication (JWT, Passport.js)
✅ E-commerce ওয়েবসাইট
✅ Blog বা CMS
✅ Real-time chat অ্যাপ (Socket.io এর সাথে)

---

## 📚 **১৩. Express.js এর সুবিধা**

| সুবিধা                  | ব্যাখ্যা                                           |
| ----------------------- | -------------------------------------------------- |
| ⚡ দ্রুত ও হালকা         | Node.js এর asynchronous nature-এর জন্য খুবই দ্রুত। |
| 🧠 শেখা সহজ             | JavaScript জানা থাকলেই শেখা যায়।                  |
| 🧩 Flexible             | ছোট থেকে বড় অ্যাপ — সব বানানো যায়।               |
| 🔌 Middleware ecosystem | প্রচুর থার্ড-পার্টি middleware আছে।                |
| 🌐 REST API ফ্রেন্ডলি   | JSON API বানানো একদম সহজ।                          |

---

## ⚠️ **১৪. Express.js এর অসুবিধা**

| অসুবিধা            | ব্যাখ্যা                                          |
| ------------------ | ------------------------------------------------- |
| ❌ Opinionated নয়  | নিজে ফোল্ডার স্ট্রাকচার ডিজাইন করতে হয়।          |
| ⚙️ Callback Hell   | Async/await ব্যবহার না করলে কোড জটিল হয়।         |
| 🧩 Built-in টুল কম | Validation, ORM ইত্যাদির জন্য আলাদা প্যাকেজ লাগে। |

---

## 🏁 **১৫. সারাংশ (সংক্ষিপ্ত টেবিল)**

| বিষয়               | বর্ণনা                               |
| ------------------ | ------------------------------------ |
| **Framework টাইপ** | Node.js এর Web Application Framework |
| **Language**       | JavaScript                           |
| **Creator**        | TJ Holowaychuk                       |
| **Release Year**   | ২০১০                                 |
| **License**        | MIT                                  |
| **Main Use**       | Server, API, এবং Backend Development |
| **Core Strength**  | Simple, Fast, and Scalable           |

---

## 🌍 **১৬. যে কোম্পানিগুলো Express.js ব্যবহার করে**

* Netflix
* Uber
* PayPal
* IBM
* Accenture
* Fox Sports

এই কোম্পানিগুলো Express.js ব্যবহার করে এর **গতি, সহজতা ও স্কেলেবলিটি**র কারণে।

---

## 🧠 **১৭. এক বাক্যে সারাংশ**

> **Express.js** হলো একটি দ্রুত, মিনিমাল, ও ফ্লেক্সিবল **Node.js ফ্রেমওয়ার্ক** যা দিয়ে সহজে **ওয়েব সার্ভার, API, ও ব্যাকএন্ড অ্যাপ্লিকেশন** তৈরি করা যায়।

---

