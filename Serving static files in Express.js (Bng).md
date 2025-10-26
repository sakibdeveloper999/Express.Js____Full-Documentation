
# 📘 Express.js-এ Static Files সার্ভ করা — পূর্ণ ডকুমেন্টেশন (বাংলা)

---

## 🧠 ১️⃣ Static Files কী?

ওয়েব অ্যাপ্লিকেশনে **Static Files** মানে হলো এমন ফাইল যেগুলো সার্ভারে পরিবর্তন হয় না, সার্ভার সেগুলো **যেমন আছে তেমনই ব্রাউজারে পাঠায়**।

উদাহরণ:

* HTML ফাইল
* CSS স্টাইলশিট
* JavaScript (client-side) ফাইল
* ইমেজ (JPG, PNG, SVG ইত্যাদি)
* ফন্ট (TTF, WOFF)
* ভিডিও, PDF ইত্যাদি

এই ফাইলগুলো **static** কারণ এগুলো সার্ভার রানটাইমে পরিবর্তন করে না — শুধু সরাসরি পাঠিয়ে দেয়।

---

## ⚙️ ২️⃣ কেন Static Files সার্ভ করতে হয়?

ওয়েব অ্যাপের ফ্রন্টএন্ডে কাজ করতে সবসময় কিছু ফাইল লাগে (HTML, CSS, JS ইত্যাদি)।
Express.js শুধু API বা backend নয়, এই static ফাইলগুলোকেও পাঠাতে পারে।

✅ ব্যবহারের ক্ষেত্র:

* ওয়েবসাইটের frontend UI তৈরি করতে।
* ছবি, আইকন, ব্যাকগ্রাউন্ড পাঠাতে।
* React/Vue এর build output সার্ভ করতে।
* কোনো ডাউনলোড ফাইল পাঠাতে (PDF, ZIP ইত্যাদি)।

---

## 🧩 ৩️⃣ `express.static()` Middleware কী?

Express.js-এ static ফাইল সার্ভ করার জন্য বিল্ট-ইন middleware আছে:

```js
express.static()
```

👉 এটা একটি ফাংশন যা **একটি নির্দিষ্ট ফোল্ডার থেকে ফাইল সার্ভ করে**।
ভিতরে এটি `serve-static` নামের লাইব্রেরি ব্যবহার করে, যা MIME type, caching, header ইত্যাদি ম্যানেজ করে।

---

## 🪄 ৪️⃣ `express.static()` কীভাবে কাজ করে (ভিতরকার প্রক্রিয়া)

ধাপে ধাপে দেখি 👇

1. তুমি একটা **directory/folder path** দাও, যেমন `public/`
2. Express সেই ফোল্ডার মনিটর করে।
3. ব্রাউজার যখন কোনো ফাইল রিকোয়েস্ট করে (যেমন `/css/style.css`),
   তখন Express ওই ফাইল খুঁজে পায়, পড়ে, এবং পাঠিয়ে দেয়।
4. যদি ফাইল না পায় —
   তাহলে `404 Not Found` পাঠায়, অথবা `next()` কল করে (যদি `fallthrough: true` সেট থাকে)।

---

## 🧱 ৫️⃣ একটি সাধারণ উদাহরণ (Step-by-Step)

### 🗂 প্রজেক্ট স্ট্রাকচার:

```
express-static-demo/
├── public/
│   ├── index.html
│   ├── css/
│   │   └── style.css
│   └── js/
│       └── app.js
└── server.js
```

### 📦 ধাপ ১: প্রজেক্ট ইনিশিয়ালাইজ

```bash
mkdir express-static-demo
cd express-static-demo
npm init -y
npm install express
```

### 🧠 ধাপ ২: server.js

```js
import express from 'express';
const app = express();

// "public" ফোল্ডার থেকে ফাইল সার্ভ করা
app.use(express.static('public'));

app.listen(3000, () => {
  console.log('Server running on http://localhost:3000');
});
```

### 💻 ধাপ ৩: index.html

```html
<!DOCTYPE html>
<html>
  <head>
    <title>Static Demo</title>
    <link rel="stylesheet" href="/css/style.css">
  </head>
  <body>
    <h1>Hello from Express Static Files!</h1>
    <script src="/js/app.js"></script>
  </body>
</html>
```

### 🎨 ধাপ ৪: style.css

```css
body {
  background-color: #f0f0f0;
  color: #333;
  font-family: Arial;
}
```

### ⚡ ধাপ ৫: app.js

```js
console.log("Static JS File Loaded!");
```

👉 এবার রান করো:

```bash
node server.js
```

ব্রাউজারে গিয়ে `http://localhost:3000` খুললে পেজ দেখাবে।

---

## 🧭 ৬️⃣ Virtual Path (Prefix) ব্যবহার

তুমি চাইলে ফোল্ডারের জন্য আলাদা URL path দিতে পারো।

```js
app.use('/static', express.static('public'));
```

এখন ফাইলগুলো সার্ভ হবে:

* `/static/index.html`
* `/static/css/style.css`

এটা শুধু একটা **virtual prefix** — ফাইল সিস্টেমে এমন ফোল্ডার নেই।

---

## ⚙️ ৭️⃣ Configuration Options (সব ব্যাখ্যা সহ)

তুমি `express.static('public', options)` এ দ্বিতীয় প্যারামিটার হিসেবে **options object** দিতে পারো।

```js
app.use(express.static('public', {
  dotfiles: 'ignore',
  etag: true,
  extensions: ['html', 'htm'],
  index: 'index.html',
  maxAge: '1d',
  redirect: true,
  fallthrough: true,
  immutable: false,
  setHeaders: function (res, path, stat) {
    res.set('x-timestamp', Date.now());
  }
}));
```

| অপশন            | কাজ                                                                                 |
| --------------- | ----------------------------------------------------------------------------------- |
| **dotfiles**    | `.` দিয়ে শুরু হওয়া ফাইল (যেমন `.env`) সার্ভ করবে কিনা (`allow`, `deny`, `ignore`) |
| **etag**        | ফাইলের জন্য cache validation header দেয়                                            |
| **extensions**  | এক্সটেনশন মিস করলে এগুলো ট্রাই করবে                                                 |
| **index**       | ডিফল্ট ফাইল, ডিরেক্টরি রিকোয়েস্ট হলে (default: `index.html`)                       |
| **maxAge**      | Cache করার সময়কাল (`'1d'`, `'2h'`, বা ms এ)                                        |
| **redirect**    | `/dir` → `/dir/` রিডিরেক্ট করবে কিনা                                                |
| **fallthrough** | ফাইল না পেলে `next()` এ যাবে কিনা                                                   |
| **immutable**   | Cache করা ফাইল পরিবর্তন হবে না বোঝায়                                               |
| **setHeaders**  | কাস্টম হেডার সেট করার ফাংশন                                                         |

---

## 🧩 ৮️⃣ Custom Header সেট করা

```js
app.use(express.static('public', {
  setHeaders: (res, path) => {
    if (path.endsWith('.html')) {
      res.setHeader('Cache-Control', 'no-cache');
    } else {
      res.setHeader('Cache-Control', 'public, max-age=31536000, immutable');
    }
  }
}));
```

---

## 💡 ৯️⃣ একাধিক ফোল্ডার সার্ভ করা

```js
app.use(express.static('public'));
app.use(express.static('assets'));
```

Express প্রথমে `public/` এ খুঁজবে, না পেলে `assets/` এ খুঁজবে।

---

## 🌍 🔟 Single Page Application (SPA) fallback

React বা Vue প্রজেক্টে সব রিকোয়েস্ট `index.html` এ দিতে হয়:

```js
import path from 'path';
import { fileURLToPath } from 'url';
const __filename = fileURLToPath(import.meta.url);
const __dirname = path.dirname(__filename);

app.use(express.static('public'));

app.get('*', (req, res) => {
  res.sendFile(path.join(__dirname, 'public', 'index.html'));
});
```

---

## 🔒 ১১️⃣ নিরাপত্তা (Security Best Practices)

✅ নিরাপদ রাখার জন্য:

* কেবল `public/` ফোল্ডারকেই এক্সপোজ করো।
* `.env`, config ফাইল ইত্যাদি কখনো সার্ভ কোরো না।
* কাস্টম path ব্যবহারে ইনপুট স্যানিটাইজ করো।
* `helmet` middleware ব্যবহার করো নিরাপত্তা হেডারের জন্য।

---

## ⚡ ১২️⃣ পারফরম্যান্স টিপস

1. **Cache Control**

   ```js
   app.use(express.static('public', { maxAge: '30d', immutable: true }));
   ```

2. **Compression ব্যবহার**

   ```js
   import compression from 'compression';
   app.use(compression());
   ```

3. **CDN/Nginx ব্যবহার**
   বড় অ্যাপের ক্ষেত্রে static ফাইল CDN বা Nginx দিয়ে সার্ভ করা দ্রুত ও স্কেলেবল।

4. **Fingerprinted filenames**
   যেমন `app.123abc.js` → নাম পরিবর্তন মানে cache refresh হবে।

---

## 🧰 ১৩️⃣ কখন Express Static ব্যবহার করব / করব না

| ব্যবহার করো যখন     | ব্যবহার করো না যখন    |
| ------------------- | --------------------- |
| ছোট/মাঝারি প্রজেক্ট | বড় স্কেল ওয়েবসাইট   |
| ডেভেলপমেন্ট স্টেজে  | হাই-ট্রাফিক প্রোডাকশন |
| ছোট অ্যাসেট         | ভিডিও/বড় ফাইল সার্ভ  |

---

## 🧮 ১৪️⃣ ভিতরের কাজের ধাপ (Internal Flow)

1. ব্রাউজার ফাইল রিকোয়েস্ট পাঠায়।
2. Express ফাইলের অস্তিত্ব চেক করে।
3. ফাইল থাকলে সেটি স্ট্রিম আকারে পাঠায়।
4. MIME type, length, cache headers সেট হয়।
5. ফাইল না পেলে `next()` বা `404` দেয়।

---

## 📋 ১৫️⃣ সম্পূর্ণ উদাহরণ (সব একসাথে)

```js
import express from 'express';
import compression from 'compression';
import path from 'path';
import { fileURLToPath } from 'url';

const app = express();
const __filename = fileURLToPath(import.meta.url);
const __dirname = path.dirname(__filename);

app.use(compression());

app.use(express.static(path.join(__dirname, 'public'), {
  maxAge: '7d',
  immutable: true,
  setHeaders: (res, filePath) => {
    if (filePath.endsWith('.html')) {
      res.setHeader('Cache-Control', 'no-cache');
    }
  }
}));

app.get('*', (req, res) => {
  res.sendFile(path.join(__dirname, 'public', 'index.html'));
});

app.listen(3000, () => console.log('Server running at http://localhost:3000'));
```

---

## 🧭 ১৬️⃣ সারাংশ

| বিষয়              | সারাংশ                              |
| ----------------- | ----------------------------------- |
| Middleware        | `express.static()`                  |
| Default Folder    | `public/`                           |
| Custom URL Prefix | `/static`                           |
| Cache Control     | `maxAge`, `immutable`, `setHeaders` |
| SPA Support       | `get('*')` fallback                 |
| Security          | সংবেদনশীল ফাইল সার্ভ না করা         |
| Performance       | CDN + compression                   |

---

## 🏁 ১৭️⃣ এক বাক্যে সারমর্ম

> Express.js-এ **Static Files সার্ভ করা** মানে হলো — `express.static()` middleware ব্যবহার করে HTML, CSS, JS, ইমেজ ইত্যাদি ফাইলগুলো সার্ভার থেকে সরাসরি ব্রাউজারে পাঠানো, যেখানে সঠিক MIME টাইপ, Cache Control, ও পারফরম্যান্স অপশন কনফিগার করা যায়।

---

