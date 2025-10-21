
---

# 🧠 ধাপ ১: Express.js এ Templating — সম্পূর্ণ বাংলা ব্যাখ্যা

---

## 📘 টেমপ্লেটিং (Templating) কী?

**Templating** মানে হলো — সার্ভার থেকে **ডাইনামিক HTML পেজ তৈরি করা** একটি **টেমপ্লেট ইঞ্জিন** ব্যবহার করে।

যখন ব্যবহারকারী ব্রাউজার থেকে অনুরোধ পাঠায়, তখন Express সার্ভার:

* একটি **template ফাইল** নেয় (যেমন `.ejs`, `.pug`, `.hbs`)
* সেটির মধ্যে **ডাটাবেস বা সার্ভার ডেটা** বসায়
* এবং তৈরি করে **চূড়ান্ত HTML পেজ**, যা ব্রাউজারে পাঠানো হয়।

অর্থাৎ, একেকটা ইউজার বা প্রোডাক্ট অনুযায়ী HTML আলাদা হবে, কিন্তু মূল ফাইল একই।

---

## 🎯 কেন Templating ব্যবহার করবো?

| কারণ                              | ব্যাখ্যা                                                                     |
| --------------------------------- | ---------------------------------------------------------------------------- |
| 🔁 **Dynamic Website**            | ডেটা অনুযায়ী পেজ তৈরি করা যায় (যেমন প্রোডাক্ট লিস্ট, ব্লগ, ইউজার প্রোফাইল) |
| ♻️ **Reusability**                | একবার হেডার–ফুটার লিখে সব পেজে ব্যবহার করা যায়                              |
| ⚡ **Server-Side Rendering (SSR)** | প্রথম লোড দ্রুত হয়, SEO ভালো হয়                                            |
| ✨ **Clean Structure**             | কোড (logic) ও HTML (view) আলাদা থাকে                                         |

---

## ⚙️ Express.js কীভাবে Templating ব্যবহার করে

Express.js নিজে HTML তৈরি করে না। এটি টেমপ্লেট ইঞ্জিন ব্যবহার করে।

পুরো প্রক্রিয়া:

1. টেমপ্লেট ইঞ্জিন ইনস্টল করা (EJS / Pug / Handlebars)
2. `app.set()` দিয়ে সেট করা
3. `/views` ফোল্ডারে `.ejs` ফাইল তৈরি করা
4. `res.render()` দিয়ে HTML রেন্ডার করা

---

## ⚒️ উদাহরণ (EJS ব্যবহার করে)

### 🧩 প্রজেক্ট স্ট্রাকচার

```
project/
 ├── app.js
 └── views/
      └── index.ejs
```

### 🧠 ধাপে ধাপে কোড

#### ১️⃣ ইনস্টল করো

```bash
npm install express ejs
```

#### ২️⃣ app.js

```js
const express = require('express');
const app = express();

// EJS সেট করা
app.set('view engine', 'ejs');

// একটি রুট তৈরি করা
app.get('/', (req, res) => {
  res.render('index', { title: 'হোম পেজ', name: 'সাকিব' });
});

// সার্ভার চালানো
app.listen(3000, () => console.log('সার্ভার চলছে http://localhost:3000'));
```

#### ৩️⃣ views/index.ejs

```ejs
<!DOCTYPE html>
<html>
<head>
  <title><%= title %></title>
</head>
<body>
  <h1>স্বাগতম, <%= name %>!</h1>
</body>
</html>
```

🔹 যখন তুমি ব্রাউজারে `http://localhost:3000` খুলো,
দেখবে — **“স্বাগতম, সাকিব!”**

---

## 🧩 Template Logic — Loop, If, Include

### ✅ লুপ (Loop)

```ejs
<ul>
  <% products.forEach(p => { %>
    <li><%= p %></li>
  <% }) %>
</ul>
```

### ✅ কন্ডিশন (If)

```ejs
<% if (loggedIn) { %>
  <p>Welcome, <%= name %></p>
<% } else { %>
  <p>Please login first.</p>
<% } %>
```

### ✅ ইনক্লুড (Include)

`views/header.ejs`

```ejs
<header><h1>My Website</h1></header>
```

`views/index.ejs`

```ejs
<%- include('header') %>
<h2>Hello <%= name %></h2>
```

---

## 🧱 Layout ব্যবহার

একটি সাধারণ layout ফাইল তৈরি করে সব পেজে ব্যবহার করা যায়।

```ejs
<html>
  <head><title><%= title %></title></head>
  <body>
    <%- body %>
  </body>
</html>
```

---

## ⚡ বাস্তব জীবনের ব্যবহার

| ব্যবহারের ক্ষেত্র     | উদাহরণ                              |
| --------------------- | ----------------------------------- |
| 📰 **Blog**           | সার্ভার থেকে পোস্ট রেন্ডার করা      |
| 🛒 **E-commerce**     | প্রোডাক্ট লিস্ট ডাইনামিকভাবে দেখানো |
| 👨‍💻 **Dashboard**   | ইউজার টেবিল রেন্ডার                 |
| 📩 **Email Template** | সার্ভার থেকে ইমেইল HTML তৈরি        |
| 📊 **Report/PDF**     | ডেটা থেকে রিপোর্ট তৈরি              |

---

## 🔒 নিরাপত্তা

1. সব সময় `<%= %>` ব্যবহার করো (HTML escape হয়)
2. `<%- %>` ব্যবহার করো শুধুমাত্র বিশ্বস্ত HTML-এর জন্য
3. ইউজার ইনপুট সরাসরি টেমপ্লেটে দিও না
4. প্রোডাকশনে caching চালাও:

   ```js
   app.set('view cache', true);
   ```

---

## ⚙️ প্রক্রিয়ার সারসংক্ষেপ

| ধাপ | কাজ                                  |
| --- | ------------------------------------ |
| 1   | টেমপ্লেট ইঞ্জিন ইনস্টল               |
| 2   | `app.set('view engine', 'ejs')`      |
| 3   | `/views` ফোল্ডারে `.ejs` ফাইল তৈরি   |
| 4   | `res.render('file', {data})` ব্যবহার |
| 5   | লুপ, কন্ডিশন, include ব্যবহার        |
| 6   | layout ও partial ব্যবহার             |
| 7   | output escape করো                    |
| 8   | caching চালাও                        |

---

## 🏁 এক কথায় সারাংশ

**Templating in Express.js** হলো এমন একটি পদ্ধতি, যেখানে সার্ভার HTML টেমপ্লেট ফাইলের মধ্যে ডেটা বসিয়ে ডাইনামিক পেজ তৈরি করে ক্লায়েন্টে পাঠায় — এতে কোড পুনঃব্যবহারযোগ্য, দ্রুত এবং SEO-ফ্রেন্ডলি ওয়েবপেজ তৈরি হয়।

---

# 🚀 ধাপ ২: EJS Product Page Demo Project

---

## 📁 ফাইল স্ট্রাকচার

```
ejs-product-demo/
 ├── app.js
 ├── package.json
 └── views/
      ├── index.ejs
      ├── product.ejs
      └── partials/
           ├── header.ejs
           └── footer.ejs
```

---

## 🧠 ধাপে ধাপে কোড

### 1️⃣ app.js

```js
const express = require('express');
const path = require('path');
const app = express();

// সেটিংস
app.set('view engine', 'ejs');
app.set('views', path.join(__dirname, 'views'));

// ডেটা (ডাটাবেস ধরো)
const products = [
  { id: 1, name: 'iPhone 15', price: 1200, desc: 'Apple flagship phone' },
  { id: 2, name: 'Samsung S24', price: 950, desc: 'Samsung high-end phone' },
  { id: 3, name: 'Google Pixel 9', price: 899, desc: 'Clean Android experience' }
];

// রুট: হোম পেজ
app.get('/', (req, res) => {
  res.render('index', { title: 'All Products', products });
});

// রুট: একক প্রোডাক্ট পেজ
app.get('/product/:id', (req, res) => {
  const product = products.find(p => p.id == req.params.id);
  if (!product) return res.status(404).send('Product not found');
  res.render('product', { title: product.name, product });
});

// সার্ভার চালু
app.listen(3000, () => console.log('Server running on http://localhost:3000'));
```

---

### 2️⃣ views/partials/header.ejs

```ejs
<header style="background:#111;color:#fff;padding:10px">
  <h1><a href="/" style="color:white;text-decoration:none;">🛒 My Product Store</a></h1>
</header>
```

---

### 3️⃣ views/partials/footer.ejs

```ejs
<footer style="background:#eee;text-align:center;padding:10px;margin-top:20px;">
  <p>© 2025 My Store. All Rights Reserved.</p>
</footer>
```

---

### 4️⃣ views/index.ejs

```ejs
<!DOCTYPE html>
<html>
<head>
  <title><%= title %></title>
</head>
<body>
  <%- include('partials/header') %>

  <h2 style="margin:10px 0;">All Products</h2>

  <ul>
    <% products.forEach(p => { %>
      <li style="margin-bottom:10px;">
        <a href="/product/<%= p.id %>"><strong><%= p.name %></strong></a> - $<%= p.price %>
      </li>
    <% }) %>
  </ul>

  <%- include('partials/footer') %>
</body>
</html>
```

---

### 5️⃣ views/product.ejs

```ejs
<!DOCTYPE html>
<html>
<head>
  <title><%= title %></title>
</head>
<body>
  <%- include('partials/header') %>

  <h2><%= product.name %></h2>
  <p><%= product.desc %></p>
  <p><strong>Price:</strong> $<%= product.price %></p>

  <a href="/">⬅ Back to Products</a>

  <%- include('partials/footer') %>
</body>
</html>
```

---

## 🧩 কীভাবে চালাবে

1️⃣ নতুন ফোল্ডার বানাও:

```bash
mkdir ejs-product-demo && cd ejs-product-demo
```

2️⃣ ইনস্টল করো:

```bash
npm init -y
npm install express ejs
```

3️⃣ উপরের কোডগুলো ফাইল অনুযায়ী লিখে রাখো।
4️⃣ রান করো:

```bash
node app.js
```

5️⃣ ব্রাউজারে যাও →
🔗 **[http://localhost:3000](http://localhost:3000)**

✅ তুমি দেখবে প্রোডাক্ট লিস্ট।
✅ কোনো প্রোডাক্টে ক্লিক করলে ডিটেইল পেজ খুলবে।

---

## 🏆 সারাংশ

এই প্রজেক্টে তুমি শিখলে —

* কীভাবে Express.js এ EJS টেমপ্লেট সেট করতে হয়
* কীভাবে ডেটা পাঠিয়ে ডাইনামিক পেজ বানানো যায়
* কীভাবে partials (header/footer) reuse করা যায়
* কীভাবে একাধিক পেজ (index & product details) যুক্ত করা যায়

---

