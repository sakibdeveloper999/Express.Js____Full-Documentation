
---

# 🧠 Express.js Full Documentation — Handling Form Data

---

## **1️⃣ Introduction**

Handling form data is one of the most fundamental parts of any web application built with Express.js.

Whenever a user submits a form (HTML form, AJAX form, or uploads files), Express needs to:

1. **Parse** the request body
2. **Extract** form fields and uploaded files
3. **Validate & sanitize** inputs
4. **Process or save** the data (database, file storage, etc.)
5. **Respond** to the client

---

## **2️⃣ Types of Form Data**

### 🧾 Common `Content-Type` Headers

| Type                                | Description                                                    | Example                 |
| ----------------------------------- | -------------------------------------------------------------- | ----------------------- |
| `application/x-www-form-urlencoded` | Default HTML form encoding                                     | Simple name=value pairs |
| `multipart/form-data`               | Used for forms that include file uploads                       | Text + Files            |
| `application/json`                  | Used when front-end (like React, Vue) sends JSON body via AJAX | Pure JSON payload       |

---

## **3️⃣ Middleware Required for Parsing Form Data**

Express.js does **not** parse incoming request bodies by default.
We use **middleware** to handle that.

### 1. `express.urlencoded()`

For traditional HTML forms (`application/x-www-form-urlencoded`)

```js
app.use(express.urlencoded({ extended: true }));
```

* `extended: true` → allows nested objects and arrays.
* Fills `req.body` with parsed key/value pairs.

### 2. `express.json()`

For APIs or AJAX requests with JSON bodies

```js
app.use(express.json());
```

### 3. `multer`

For forms that send **files** (`multipart/form-data`)

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

## **4️⃣ Basic Example (No Files)**

### 🗂 Project Structure

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
  <input type="text" name="name" placeholder="Enter name" required>
  <input type="email" name="email" placeholder="Enter email" required>
  <button type="submit">Submit</button>
</form>
```

---

## **5️⃣ Handling File Uploads (multipart/form-data)**

```bash
npm install multer
```

### 🧩 index.mjs

```js
import express from 'express';
import multer from 'multer';

const app = express();

// Configure storage
const storage = multer.diskStorage({
  destination: (req, file, cb) => cb(null, 'uploads/'),
  filename: (req, file, cb) =>
    cb(null, Date.now() + '-' + file.originalname)
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

## **6️⃣ Using AJAX (FormData + fetch)**

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

## **7️⃣ Handling Validation**

Install `express-validator`:

```bash
npm install express-validator
```

Example:

```js
import { body, validationResult } from 'express-validator';

app.post('/register',
  body('email').isEmail().withMessage('Invalid email'),
  body('password').isLength({ min: 6 }).withMessage('Password too short'),
  (req, res) => {
    const errors = validationResult(req);
    if (!errors.isEmpty()) return res.status(400).json({ errors: errors.array() });
    res.send('User registered!');
  }
);
```

---

## **8️⃣ Nested and Array Data**

```html
<input name="user[name]" value="Sakib">
<input name="user[email]" value="sakib@mail.com">
<input name="skills[]" value="JS">
<input name="skills[]" value="Express">
```

When using `express.urlencoded({ extended: true })`:

```js
req.body = {
  user: { name: 'Sakib', email: 'sakib@mail.com' },
  skills: ['JS', 'Express']
}
```

---

## **9️⃣ Common Validation Patterns**

| Validation     | Example Code                                      |
| -------------- | ------------------------------------------------- |
| Required field | `body('name').notEmpty()`                         |
| Minimum length | `body('username').isLength({ min: 3 })`           |
| Email format   | `body('email').isEmail()`                         |
| Number check   | `body('age').isInt({ min: 1 })`                   |
| Custom logic   | `body('username').custom(val => val !== 'admin')` |

---

## **🔟 Error Handling**

Global error handler:

```js
app.use((err, req, res, next) => {
  console.error(err);
  res.status(500).json({ message: err.message });
});
```

For Multer:

```js
app.use((err, req, res, next) => {
  if (err instanceof multer.MulterError)
    return res.status(400).send({ message: err.message });
  next(err);
});
```

---

## **11️⃣ Security Recommendations**

| Risk                       | Solution                                          |
| -------------------------- | ------------------------------------------------- |
| Oversized uploads          | Use `limits` in multer and `limit` in body parser |
| Dangerous file types       | Restrict MIME types + validate extensions         |
| Stored XSS / SQL Injection | Sanitize and escape inputs                        |
| CSRF attacks               | Use CSRF tokens for browser forms                 |
| Sensitive uploads          | Store outside public folder                       |
| Helmet                     | Use `helmet()` middleware                         |
| HTTPS                      | Always use TLS in production                      |

---

## **12️⃣ Advanced Use Cases**

### 🧰 Upload to Cloud (e.g., AWS S3)

* Use multer-s3 or stream file directly to S3 bucket.
* Example:

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

### 🧰 Handle Large Data Streams

* Use `busboy` or `formidable` if you need streaming for huge uploads.

---

## **13️⃣ Summary Flow**

```
[HTML Form] → [Express Middleware]
    → express.urlencoded() / multer / express.json()
        → req.body + req.file(s)
            → Validation (express-validator)
                → Save to DB or storage
                    → Send response
```

---

## **14️⃣ Common Mistakes**

❌ Missing `enctype="multipart/form-data"` → file not sent
❌ Forgetting `express.urlencoded()` → `req.body` undefined
❌ Using wrong input name → `req.file` empty
❌ Not setting storage destination → multer error
❌ Serving upload folder publicly → security risk

---

## **15️⃣ Best Practice Checklist**

✅ Use both `express.json()` & `express.urlencoded()`
✅ Use Multer for file parsing
✅ Validate & sanitize all user input
✅ Limit body size & file upload size
✅ Never store files directly inside `/public`
✅ Handle errors gracefully
✅ Secure the app with Helmet, CSRF, and HTTPS

---

## **16️⃣ Conclusion**

Handling form data in Express.js means combining:

1. Correct **middleware** (`express.urlencoded`, `express.json`, `multer`)
2. Strong **validation & sanitization**
3. Proper **error handling & security**
4. Clean **folder structure**
5. Consistent **input naming and encoding**

Once mastered, you can handle everything from **simple contact forms** to **complex file upload APIs** safely and efficiently.

---
## **17 Youtube**

```
<iframe width="560" height="315" src="https://www.youtube-nocookie.com/embed/CNJnrKkTjKo?si=4kClvJQQa24ibail" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>
```
