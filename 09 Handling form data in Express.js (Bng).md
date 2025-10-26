
---

# 🧠 Express.js ফর্ম ডেটা হ্যান্ডলিং সম্পূর্ণ ডকুমেন্টেশন (Bangla)

---

## **1️⃣ পরিচিতি**

Express.js-এ ফর্ম ডেটা হ্যান্ডলিং ওয়েব অ্যাপের সবচেয়ে গুরুত্বপূর্ণ অংশগুলোর একটি।

যখন ইউজার একটি ফর্ম সাবমিট করে (HTML ফর্ম, AJAX ফর্ম, বা ফাইল আপলোড), তখন Express-কে করতে হয়:

1. **রিকোয়েস্ট বডি পার্স করা**
2. **ফর্ম ফিল্ড ও আপলোডকৃত ফাইল বের করা**
3. **ভ্যালিডেশন ও স্যানিটাইজেশন**
4. **ডেটা প্রসেস বা সংরক্ষণ** (ডাটাবেস/ফাইল স্টোরেজ)
5. **ক্লায়েন্টকে রেসপন্স পাঠানো**

---

## **2️⃣ ফর্ম ডেটার ধরন**

### 🧾 সাধারণ `Content-Type` হেডার

| টাইপ                                | বর্ণনা                          | উদাহরণ             |
| ----------------------------------- | ------------------------------- | ------------------ |
| `application/x-www-form-urlencoded` | ডিফল্ট HTML ফর্ম এনকোডিং        | সরল key=value জোড়া |
| `multipart/form-data`               | ফর্মে ফাইল থাকলে                | টেক্সট + ফাইল      |
| `application/json`                  | AJAX/SPA থেকে JSON পাঠানোর জন্য | পিওর JSON          |

---

## **3️⃣ ফর্ম ডেটা পার্স করার জন্য Middleware**

Express ডিফল্টে ইনকামিং রিকোয়েস্ট বডি পার্স করে না।
আমরা **Middleware** ব্যবহার করি:

### 1. `express.urlencoded()`

সাধারণ HTML ফর্মের জন্য

```js
app.use(express.urlencoded({ extended: true }));
```

* `extended: true` → নেস্টেড অবজেক্ট ও অ্যারে পার্স করা যায়
* `req.body`-তে Key/Value পায়

### 2. `express.json()`

AJAX বা API JSON রিকোয়েস্টের জন্য

```js
app.use(express.json());
```

### 3. `multer`

ফাইল আপলোডের জন্য (`multipart/form-data`)

```bash
npm install multer
```

```js
import multer from 'multer';

const upload = multer({ dest: 'uploads/' });
app.post('/upload', upload.single('file'), (req, res) => {
  console.log(req.file);
  console.log(req.body);
});
```

---

## **4️⃣ বেসিক উদাহরণ (ফাইল ছাড়া)**

### 🗂 প্রজেক্ট স্ট্রাকচার

```
form-demo/
├─ index.mjs
└─ public/
   └─ form.html
```

### 🧩 index.mjs

```js
import express from 'express';
import path from 'path';
import { fileURLToPath } from 'url';

const __filename = fileURLToPath(import.meta.url);
const __dirname = path.dirname(__filename);

const app = express();
app.use(express.urlencoded({ extended: true }));
app.use(express.json());

app.use(express.static(path.join(__dirname, 'public')));

app.post('/submit-form', (req, res) => {
  console.log(req.body);
  res.send(`Received: ${JSON.stringify(req.body)}`);
});

app.listen(3000, () => console.log('Server running on http://localhost:3000'));
```

### 🧩 public/form.html

```html
<form action="/submit-form" method="post">
  <input type="text" name="name" placeholder="নাম লিখুন" required>
  <input type="email" name="email" placeholder="ইমেইল লিখুন" required>
  <button type="submit">Submit</button>
</form>
```

---

## **5️⃣ ফাইল আপলোড হ্যান্ডলিং (`multipart/form-data`)**

```bash
npm install multer
```

### 🧩 index.mjs

```js
import express from 'express';
import multer from 'multer';

const app = express();

// স্টোরেজ কনফিগারেশন
const storage = multer.diskStorage({
  destination: (req, file, cb) => cb(null, 'uploads/'),
  filename: (req, file, cb) => cb(null, Date.now() + '-' + file.originalname)
});

const upload = multer({ storage });

// Single file
app.post('/upload-single', upload.single('avatar'), (req, res) => {
  res.send({
    textFields: req.body,
    file: req.file
  });
});

// Multiple files
app.post('/upload-multi', upload.array('photos', 5), (req, res) => {
  res.send({
    textFields: req.body,
    files: req.files
  });
});

app.listen(3000, () => console.log('Server started on http://localhost:3000'));
```

### 🧩 HTML Form

```html
<form action="/upload-single" method="post" enctype="multipart/form-data">
  <input type="text" name="username" required>
  <input type="file" name="avatar" required>
  <button>Upload</button>
</form>
```

---

## **6️⃣ AJAX ফর্ম (FormData + fetch)**

```html
<form id="ajaxForm">
  <input name="username" placeholder="Username" required>
  <input type="file" name="avatar">
  <button>Send via JS</button>
</form>

<script>
const form = document.getElementById('ajaxForm');
form.addEventListener('submit', async (e) => {
  e.preventDefault();
  const fd = new FormData(form);
  const res = await fetch('/upload-single', { method: 'POST', body: fd });
  const json = await res.json();
  console.log(json);
});
</script>
```

---

## **7️⃣ ভ্যালিডেশন হ্যান্ডলিং**

Install `express-validator`:

```bash
npm install express-validator
```

```js
import { body, validationResult } from 'express-validator';

app.post('/register',
  body('email').isEmail().withMessage('ভ্যালিড ইমেইল দিন'),
  body('password').isLength({ min: 6 }).withMessage('পাসওয়ার্ড কমপক্ষে ৬ অক্ষরের হতে হবে'),
  (req, res) => {
    const errors = validationResult(req);
    if (!errors.isEmpty()) return res.status(400).json({ errors: errors.array() });
    res.send('User registered!');
  }
);
```

---

## **8️⃣ Nested এবং Array ডেটা**

```html
<input name="user[name]" value="Sakib">
<input name="user[email]" value="sakib@mail.com">
<input name="skills[]" value="JS">
<input name="skills[]" value="Express">
```

```js
req.body = {
  user: { name: 'Sakib', email: 'sakib@mail.com' },
  skills: ['JS', 'Express']
}
```

---

## **9️⃣ সাধারণ ভ্যালিডেশন প্যাটার্ন**

| ভ্যালিডেশন     | উদাহরণ কোড                                        |
| -------------- | ------------------------------------------------- |
| Required       | `body('name').notEmpty()`                         |
| Minimum length | `body('username').isLength({ min: 3 })`           |
| Email          | `body('email').isEmail()`                         |
| Number check   | `body('age').isInt({ min: 1 })`                   |
| Custom logic   | `body('username').custom(val => val !== 'admin')` |

---

## **🔟 এরর হ্যান্ডলিং**

```js
app.use((err, req, res, next) => {
  console.error(err);
  res.status(500).json({ message: err.message });
});
```

Multer এরর হ্যান্ডলিং:

```js
app.use((err, req, res, next) => {
  if (err instanceof multer.MulterError)
    return res.status(400).send({ message: err.message });
  next(err);
});
```

---

## **11️⃣ নিরাপত্তা সুপারিশ**

| ঝুঁকি              | সমাধান                                            |
| ------------------ | ------------------------------------------------- |
| বড় ফাইল আপলোড      | multer এবং body parser-এ size limits ব্যবহার করুন |
| বিপজ্জনক ফাইল টাইপ | MIME type এবং extension চেক করুন                  |
| XSS / SQL ইনজেকশন  | ইনপুট স্যানিটাইজ করুন এবং escape করুন             |
| CSRF               | ব্রাউজার ফর্মে CSRF টোকেন ব্যবহার করুন            |
| সেনসিটিভ ফাইল      | public ফোল্ডারের বাইরে আপলোড রাখুন                |
| Helmet             | `helmet()` middleware ব্যবহার করুন                |
| HTTPS              | production-এ TLS ব্যবহার করুন                     |

---

## **12️⃣ উন্নত ব্যবহার**

### ক্লাউডে আপলোড (AWS S3)

```js
import multerS3 from 'multer-s3';
import AWS from 'aws-sdk';
const s3 = new AWS.S3();
const upload = multer({
  storage: multerS3({
    s3,
    bucket: 'your-bucket',
    key: (req, file, cb) => cb(null, Date.now() + '-' + file.originalname)
  })
});
```

### বড় ডেটা স্ট্রিমিং

* বড় ফাইলের জন্য `busboy` বা `formidable` ব্যবহার করুন

---

## **13️⃣ সারসংক্ষেপ ফ্লো**

```
[HTML Form] → [Express Middleware]
    → express.urlencoded() / multer / express.json()
        → req.body + req.file(s)
            → Validation
                → Save to DB or storage
                    → Send response
```

---

## **14️⃣ সাধারণ ভুল**

❌ `enctype="multipart/form-data"` ভুল হলে ফাইল যাবে না
❌ `express.urlencoded()` না ব্যবহার করলে `req.body` undefined
❌ ভুল input name → `req.file` empty
❌ storage destination না দিলে multer error
❌ upload folder public-এ রাখলে নিরাপত্তা ঝুঁকি

---

## **15️⃣ Best Practice Checklist**

✅ `express.json()` ও `express.urlencoded()` উভয় ব্যবহার করুন
✅ ফাইল আপলোডের জন্য Multer ব্যবহার করুন
✅ সব ইনপুট validate ও sanitize করুন
✅ বডি এবং ফাইল সাইজ limit দিন
✅ ফাইল `/public`-এ সরাসরি রাখবেন না
✅ এরর Gracefully হ্যান্ডেল করুন
✅ Helmet, CSRF, HTTPS ব্যবহার করুন

---

## **16️⃣ উপসংহার**

Express.js-এ ফর্ম ডেটা হ্যান্ডলিং মানে হলো:

1. সঠিক **middleware** ব্যবহার করা (`express.urlencoded`, `express.json`, `multer`)
2. শক্তিশালী **validation & sanitization**
3. সঠিক **error handling & security**
4. পরিচ্ছন্ন **ফোল্ডার স্ট্রাকচার**
5. সঠিক **input naming এবং encoding**

একবার দক্ষ হয়ে গেলে, তুমি সহজ **contact form** থেকে শুরু করে **complex file upload API** পর্যন্ত সবকিছু সঠিকভাবে ও নিরাপদে হ্যান্ডেল করতে পারবে।

---

