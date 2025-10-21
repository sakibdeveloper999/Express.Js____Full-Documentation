ঠিক আছে! এবার আমি পুরো EJS ডকুমেন্টেশনটিকে **বাংলায়** রূপান্তর করছি। সব তথ্য, উদাহরণ, ও কোড স্ট্রাকচার ঠিক রাখব, শুধু ভাষা পরিবর্তন হবে।

---

# EJS — সম্পূর্ণ ডকুমেন্টেশন ও ব্যবহারিক গাইড (ধাপে ধাপে)

---

## EJS কী?

EJS (Embedded JavaScript) হল একটি লাইটওয়েট টেমপ্লেটিং ভাষা যা HTML এর ভিতরে JavaScript কোড এম্বেড করার মাধ্যমে সার্ভার-সাইডে HTML তৈরি করতে সাহায্য করে। এটি সরল এবং দ্রুত। EJS সাধারণত Express এর সাথে ব্যবহার করা হয় ডাইনামিক পেজ রেন্ডার করার জন্য।

**মূল বৈশিষ্ট্য:**

* খুব সহজ সিনট্যাক্স (JavaScript জানা থাকলে শেখা সহজ)।
* ডিফল্টভাবে HTML এস্কেপিং (XSS থেকে নিরাপদ)।
* লাইব্রেরি বা Express ভিউ ইঞ্জিন হিসেবে ব্যবহার করা যায়।
* হালকা এবং দ্রুত — সার্ভার-সাইড HTML, ইমেইল টেমপ্লেট, অ্যাডমিন UI-এর জন্য উপযুক্ত।

---

## মূল ধারণা

* **ভিউ/টেমপ্লেট:** `.ejs` ফাইল যা HTML এবং JS কোড ধারণ করে।
* **ডাটা/লোকালস:** টেমপ্লেটে ডাইনামিক মান পাঠানোর জন্য অবজেক্ট।
* **রেন্ডারিং:** টেমপ্লেট + ডাটা → HTML।
* **পার্শিয়ালস/ইনক্লুডস:** পুনরায় ব্যবহারযোগ্য টেমপ্লেট অংশ (header/footer)।
* **এস্কেপিং:** EJS ডিফল্টভাবে আউটপুট এ HTML এস্কেপিং করে।

---

## EJS সিনট্যাক্স (প্রধান ট্যাগ)

* **এস্কেপড আউটপুট (নিরাপদ):**

```ejs
<%= variable %>
```

* **আনএস্কেপড আউটপুট (raw HTML):**

```ejs
<%- htmlString %>
```

* **JS কোড রান (কোনও সরাসরি আউটপুট নয়):**

```ejs
<% if (user) { %>
  Hello
<% } %>
```

* **টেমপ্লেট কমেন্ট (রেন্ডারিং এ আসে না):**

```ejs
<% /* comment */ %>
```

* **লুপ/কন্ট্রোল ফ্লো:**

```ejs
<% for (let i = 0; i < items.length; i++) { %>
  <li><%= items[i] %></li>
<% } %>
```

**মনে রাখবেন:**

* `<%= %>` সবসময় HTML এস্কেপ করে → XSS থেকে নিরাপদ।
* `<%- %>` ব্যবহার করবেন শুধুমাত্র নিরাপদ HTML এর জন্য।

---

## ইনস্টলেশন ও Express-এ বেসিক সেটআপ

1. প্রজেক্ট তৈরি ও ইনস্টল:

```bash
npm init -y
npm install express ejs
```

2. Express কনফিগার করা:

```js
const express = require('express');
const app = express();

// EJS ভিউ ইঞ্জিন হিসেবে সেট করা
app.set('view engine', 'ejs');

app.get('/', (req, res) => {
  res.render('index', { title: 'Home', user: { name: 'Sakib' }});
});

app.listen(3000);
```

3. উদাহরণ `views/index.ejs`:

```ejs
<!doctype html>
<html>
<head>
  <title><%= title %></title>
</head>
<body>
  <h1>স্বাগতম, <%= user.name %>!</h1>
</body>
</html>
```

`res.render(viewName, locals)` → নির্দিষ্ট টেমপ্লেট খুঁজে রেন্ডার করে HTML পাঠায়।

---

## EJS লাইব্রেরি API

EJS Express ছাড়া ব্যবহার করা যায়:

* `ejs.render(templateString, data, options)` → টেমপ্লেট স্ট্রিং রেন্ডার করে HTML দেয়।
* `ejs.renderFile(filename, data, options, callback)` → ফাইল রেন্ডার (অ্যাসিঙ্ক)।
* `ejs.compile(templateString, options)` → টেমপ্লেট ফাংশন কম্পাইল করে পুনঃব্যবহারযোগ্য।

**উদাহরণ:**

```js
const ejs = require('ejs');

const tpl = 'Hello <%= name %>!';
const html = ejs.render(tpl, { name: 'Sakib' });
// html === 'Hello Sakib!'

const fn = ejs.compile('Sum: <%= a + b %>');
console.log(fn({ a: 1, b: 2 })); // 'Sum: 3'
```

---

## গুরুত্বপূর্ণ অপশন

* `filename` → স্ট্যাক ট্রেস ও include এর জন্য।
* `cache` → টেমপ্লেট কেচিং (true/false)।
* `delimiter` → ট্যাগের ডিলিমিটার পরিবর্তন।
* `rmWhitespace` → টেমপ্লেটের অতিরিক্ত স্পেস/নিউলাইন সরায়।
* `compileDebug` → development এ stack trace সহজ।
* `async` → অ্যাসিঙ্ক্রোনাস টেমপ্লেট (সাপোর্ট আছে কি না সংস্করণে)।

---

## পার্শিয়ালস ও include

```ejs
<%- include('partials/header') %>
<h1>Page body</h1>
<%- include('partials/footer') %>
```

* `include()` → টেমপ্লেটের অংশ ইনসার্ট করে।
* `include()` সাধারণত `<%- %>` দিয়ে ব্যবহার করা হয়।
* parent locals include-এ পাওয়া যায়।

---

## লেআউট প্যাটার্ন

### ১) ম্যানুয়াল লেআউট

```js
const body = ejs.renderFile('views/posts.ejs', { posts });
const html = ejs.renderFile('views/layout.ejs', { body });
```

### ২) Partial-based approach

```ejs
<%- include('partials/header') %>
<main> ... </main>
<%- include('partials/footer') %>
```

---

## Helpers: `app.locals` ও `res.locals`

* `app.locals` → গ্লোবাল হেল্পার:

```js
app.locals.siteName = 'My Site';
app.locals.formatDate = d => d.toLocaleDateString();
```

* `res.locals` → রিকোয়েস্ট স্পেসিফিক হেল্পার:

```js
app.use((req, res, next) => {
  res.locals.currentUser = req.user;
  next();
});
```

---

## এস্কেপিং ও নিরাপত্তা

* `<%= %>` ডিফল্টভাবে HTML এস্কেপ করে → XSS নিরাপদ।
* `<%- %>` শুধুমাত্র নিরাপদ HTML এর জন্য।
* CSP এবং অন্যান্য HTTP হেডার ব্যবহার করুন।
* ইউজার-ইনপুট সবসময় সার্ভার-সাইডে ভ্যালিডেট করুন।

---

## পারফরম্যান্স ও কেচিং

* প্রোডাকশনে টেমপ্লেট কেচিং ব্যবহার করুন।
* কমপ্লেক্স/ওজনদার লুপ টেমপ্লেটে রাখবেন না।
* `ejs.compile` ব্যবহার করে একবার কম্পাইল করে পুনঃব্যবহার করতে পারেন।

---

## ডিবাগিং টিপস

* `filename` ব্যবহার করে include ও stack trace সহজ করুন।
* `compileDebug: true` development এ।
* টেমপ্লেট ছোট ছোট করে পরীক্ষা করুন।

---

## অ্যাডভান্সড টপিকস

1. **ক্লায়েন্ট-সাইড রেন্ডারিং:** EJS কম্পাইল করে ক্লায়েন্টে পাঠানো যায়।
2. **Async helper & include:** কিছু EJS সংস্করণ async টেমপ্লেট সাপোর্ট করে।
3. **কাস্টম ডিলিমিটার:** `<% %>` conflicts এ পরিবর্তন করা যায়।
4. **include path resolution:** `filename` বা `root/views` ঠিক করে।

---

## সাধারণ সমস্যাগুলি

* `<%- %>` ব্যবহার করে untrusted input → XSS।
* ভুল locals পাঠানো → undefined।
* ভুল views path → template not found।
* টেমপ্লেটে ভারী logic → maintainability কম।

---

## টেস্টিং টেমপ্লেট

```js
const assert = require('assert');
const ejs = require('ejs');
const tpl = '<h1><%= title %></h1>';

const html = ejs.render(tpl, { title: 'Test' });
assert(html.includes('<h1>Test</h1>'));
```

---

## উদাহরণ: পূর্ণ প্রজেক্ট (Runnable)

**ফোল্ডার স্ট্রাকচার**

```
ejs-doc-demo/
├─ app.js
├─ package.json
├─ views/
│  ├─ index.ejs
│  ├─ products.ejs
│  └─ partials/
│     ├─ header.ejs
│     └─ footer.ejs
└─ public/
   └─ css/styles.css
```

**app.js**

```js
const express = require('express');
const path = require('path');
const app = express();
const port = 3000;

app.set('view engine', 'ejs');
app.set('views', path.join(__dirname, 'views'));
app.use(express.static(path.join(__dirname, 'public')));

app.locals.siteName = 'EJS Demo Store';
app.locals.formatCurrency = v => '$' + Number(v).toFixed(2);

app.use((req, res, next) => {
  res.locals.currentPath = req.path;
  res.locals.user = { name: 'Guest' };
  next();
});

app.get('/', (req, res) => res.render('index', { title: 'Welcome' }));

app.get('/products', (req, res) => {
  const products = [
    { id: 1, name: 'T-shirt', price: 19.99 },
    { id: 2, name: 'Sneakers', price: 79.99 },
    { id: 3, name: 'Hat', price: 12.5 }
  ];
  res.render('products', { title: 'Products', products });
});

app.listen(port, () => console.log(`Listening on http://localhost:${port}`));
```

**partials/header.ejs**

```ejs
<!doctype html>
<html>
<head>
  <meta charset="utf-8">
  <title><%= title ? title + ' — ' + siteName : siteName %></title>
  <link rel="stylesheet" href="/css/styles.css">
</head>
<body>
  <header>
    <h1><a href="/"><%= siteName %></a></h1>
    <nav>
      <a href="/" class="<%= currentPath === '/' ? 'active' : '' %>">Home</a>
      <a href="/products" class="<%= currentPath === '/products' ? 'active' : '' %>">Products</a>
    </nav>
  </header>
  <main>
```

**partials/footer.ejs**

```ejs
  </main>
  <footer>
    <small>&copy; <%= new Date().getFullYear() %> <%= siteName %></small>
  </footer>
</body>
</html>
```

**index.ejs**

```ejs
<%- include('partials/header') %>
<section>
  <h2><%= title %></h2>
  <p>Hello, <%= user.name %> — এই হোমপেজটি EJS দিয়ে রেন্ডার করা হয়েছে।</p>
</section>
<%- include('partials/footer') %>
```

**products.ejs**

```ejs
<%- include('partials/header') %>
<section>
  <h2><%= title %></h2>
  <ul>
    <% if (products.length) { %>
      <% products.forEach(p => { %>
        <li><strong><%= p.name %></strong> — <%= formatCurrency(p.price) %></li>
      <% }) %>
    <% } else { %>
      <li>কোনো প্রোডাক্ট পাওয়া যায়নি।</li>
    <% } %>
  </ul>
</section>
<%- include('partials/footer') %>
```

**public/css/styles.css**

```css
body { font-family: system-ui, sans-serif; margin: 0; padding: 0; }
header { background:#222; color:#fff; padding: 1rem; }
header a { color: #fff; text-decoration: none; margin-right: 1rem; }
main { padding: 1rem; }
.active { text-decoration: underline; }
```

---

