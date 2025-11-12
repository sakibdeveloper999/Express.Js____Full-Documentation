# ১. সংক্ষেপে — কুকি কি?

কুকি হলো ছোট একটি ডেটা-পিস যা সার্ভার ব্রাউজারকে পাঠিয়ে রাখে এবং পরবর্তীতে ঐ ব্রাউজার একই সার্ভারে অনুরোধ পাঠালে ব্রাউজার সেটা ফেরত পাঠায়। সার্ভার `Set-Cookie` হেডারে কুকি সেট করে, এবং ক্লায়েন্ট পরবর্তী অনুরোধে `Cookie` হেডারে সেটা পাঠায়।

# ২. সাধারণত যা লাগবে — ইনস্টলেশন

`cookie-parser` লাইব্রেরি খুবই সাধারণ এবং উপকারী — এটি ইনকামিং কুকি পার্স করে ও সাইনড কুকি সাপোর্ট করে।

```bash
npm install express cookie-parser
# অথবা yarn ব্যবহার করলে
yarn add express cookie-parser
```

# ৩. মিনিমাল Express (ES modules) সেটআপ

```js
// index.mjs বা server.mjs (ES module)
import express from 'express';
import cookieParser from 'cookie-parser';

const app = express();
const PORT = 3000;

// সাইনড কুকি ব্যবহার করতে চাইলে সিক্রেট দিন, না চাইলে বাদ দিন।
app.use(cookieParser('my-secret-string'));

app.use(express.json());

app.get('/', (req, res) => {
  res.send('Hello');
});

app.listen(PORT, () => console.log(`Listening ${PORT}`));
```

# ৪. সার্ভার থেকে কুকি সেট করা

কুকি সেট করতে `res.cookie(name, value, options)` ব্যবহার করেন।

```js
app.get('/set-cookie', (req, res) => {
  // বেসিক কুকি
  res.cookie('theme', 'dark', { maxAge: 1000 * 60 * 60 * 24 }); // ১ দিন

  // HttpOnly কুকি (ক্লায়েন্ট সাইড JS থেকে অ্যাক্সেস করা যায় না) — auth টোকেনের জন্য ভালো
  res.cookie('sessionId', 'abc123', {
    httpOnly: true,
    secure: process.env.NODE_ENV === 'production', // প্রোডাকশনে HTTPS-only
    maxAge: 1000 * 60 * 60 * 24 * 7, // ৭ দিন
    sameSite: 'lax'
  });

  // সাইনড কুকি
  res.cookie('signedName', 'sakib', { signed: true, httpOnly: true });

  res.send('Cookies set');
});
```

**গুরুত্বপূর্ণ কুকি অপশনসমূহ**

* `httpOnly: true` — ক্লায়েন্ট সাইড `document.cookie` থেকে দেখা/এডিট করা যাবে না (XSS থেকে কুকি চুরি রোধে সহায়ক)।
* `secure: true` — কেবল HTTPS-এ পাঠানো হবে।
* `maxAge` (মিলিসেকেন্ড) বা `expires` (Date) — কুকির মেয়াদ।
* `sameSite: 'lax' | 'strict' | 'none'` — ক্রস-সাইট অনুরোধে কুকি পাঠানোর নিয়ন্ত্রণ। `none` ব্যবহার করলে `secure: true` থাকা বাধ্যতামূলক।
* `path` এবং `domain` — কুকির স্কোপ সীমিত করতে ব্যবহৃত।
* `signed: true` — `cookie-parser`-এ সিক্রেট দিলে কুকি টেম্পারিং চেক করা যায়।

# ৫. রিকোয়েস্ট হ্যান্ডলার-এ কুকি পড়া

`cookie-parser` ব্যবহার করলে `req.cookies` (অসাইনড কুকি) ও `req.signedCookies` (সাইনড কুকি) পাওয়া যায়।

```js
app.get('/read', (req, res) => {
  console.log(req.cookies);            // সব unsigned কুকি
  console.log(req.signedCookies);      // সব signed কুকি
  const theme = req.cookies.theme;
  res.json({ theme });
});
```

# ৬. কুকি মুছা/ডিলিট করা

`res.clearCookie(name, options?)` ব্যবহার করুন। যদি কুকি সেট করার সময় `path`/`domain` অ্যাপ্লাই করা হয়ে থাকে, তখন সেগুলো মেলাতে হবে।

```js
app.get('/logout', (req, res) => {
  res.clearCookie('sessionId'); // কুকি সরিয়ে দেয়
  res.send('Logged out');
});
```

# ৭. সাইনড কুকি — উদাহরণ

```js
// cookieParser('my-secret-string') দিয়ে শুরু করা আছে ধরে নিচ্ছি
app.get('/set-signed', (req, res) => {
  res.cookie('user', 'mdsakib', { signed: true, httpOnly: true });
  res.send('Signed cookie set');
});
app.get('/check-signed', (req, res) => {
  res.send({ signed: req.signedCookies.user || null });
});
```

# ৮. অথেনটিকেশনের জন্য কুকি ব্যবহারের প্যাটার্ন

* **সার্ভার-সাইড সেশন (session id):** কুকিতে শুধু session id রাখেন (HttpOnly)। সার্ভারে সেশন ডেটা রাখা হয় — মেমরি/Redis/ডাটাবেজ ইত্যাদি। `express-session` বা কাস্টম সেশন মেকানিজম ব্যবহার করা হয়।
* **JWT কুকিতে রাখা:** Access/Refresh টোকেন `HttpOnly, Secure` কুকিতে রাখতে পারেন। Access টোকেন ছোট মেয়াদের রাখুন, refresh টোকেন HttpOnly-তে রাখুন এবং রোটেশন/রিফ্রেশ প্যাটার্ন ব্যবহার করুন।
* **Double-submit CSRF প্রতিরোধ:** সার্ভার একটি কুকি সেট করে এবং ক্লায়েন্ট আলাদা হেডার/বডিতে একই মান পাঠায় — সার্ভার মিল চেক করে।

Auth কুকির উদাহরণ (HttpOnly + Secure):

```js
res.cookie('access_token', token, {
  httpOnly: true,
  secure: true,       // প্রোডাকশনে HTTPS লাগবে
  sameSite: 'lax',
  maxAge: 1000 * 60 * 15 // ১৫ মিনিট
});
```

# ৯. CORS ও ক্রস-সাইট কুকি (খুব গুরুত্বপূর্ণ)

ফ্রন্টেন্ড যদি আলাদা origin-এ থাকে, তাহলে ব্রাউজার কুকি পাঠাবে কেবল যখন:

* ক্লায়েন্ট সাইড ফেচে `credentials: 'include'` (অথবা same-origin হলে `credentials: 'same-origin'`) ব্যবহার করা হবে,
* সার্ভার রেসপন্সে `Access-Control-Allow-Credentials: true` থাকবে,
* সার্ভারের `Access-Control-Allow-Origin` `*` নয়, নির্দিষ্ট origin (যেমন `https://app.example.com`) হবে,
* ক্রস-সাইট কুকি হলে সাধারণত `sameSite: 'none'` এবং `secure: true` লাগবে (অর্থাৎ HTTPS)।

সার্ভার সাইড `cors` কনফিগ উদাহরণ:

```js
import cors from 'cors';

app.use(cors({
  origin: 'https://app.example.com',
  credentials: true,
}));

// ক্লায়েন্ট সাইড fetch:
fetch('https://api.example.com/data', {
  method: 'GET',
  credentials: 'include'
});
```

# ১০. ক্লায়েন্ট থেকে কুকি সেট করা (JavaScript)

* `document.cookie` দিয়ে কুকি সেট করা যায় (কিন্তু auth কুকিগুলো HttpOnly হলে JS থেকে সেট/পড়া যাবে না — সেটা নীরিক্ষিত সুবিধা জন্যই ভাল)։

```js
document.cookie = "name=val;path=/;max-age=3600";
```

* ক্রস-ওরিজিন অনুরোধে কুকি পাঠাতে `fetch`-এ `credentials: 'include'` ব্যবহার করুন:

```js
fetch('https://api.example.com/login', {
  method: 'POST',
  credentials: 'include',
  body: JSON.stringify({ username, password }),
  headers: { 'Content-Type': 'application/json' }
});
```

# ১১. সিকিউরিটি বেস্ট প্র্যাকটিস

* সংবেদনশীল টোকেন `HttpOnly` রাখুন যাতে JS থেকে ধরা না যায় (XSS রোধে সহায়ক)।
* প্রোডাকশনে `Secure: true` ব্যবহার করুন (HTTPS বাধ্যতামূলক)।
* `SameSite`— `Lax` বা `Strict` ব্যবহার করে CSRF ঝুঁকি কমান; যদি তৃতীয় পক্ষ কুকি দরকার হয় তাহলে `None` ব্যবহার করুন কিন্তু অবশ্যই `Secure` লাগবে।
* কুকিতে বড় ডেটা (বড় JSON) রাখবেন না; যতটা সম্ভব কম ডেটা রাখুন।
* ক্লায়েন্ট-সাইডে গুরুত্বপূর্ণ ডেটা রাখা হলে সাইনড কুকি বা HMAC ব্যবহার করুন যাতে টেম্পার চেক করা যায়।
* সংবেদনশীল স্টেট সার্ভারে রাখুন (session id কুকিতে রাখুন)।
* CSRF প্রোটেকশন যোগ করুন: `sameSite`, CSRF টোকেন, অথবা double-submit কুকি প্যাটার্ন।

# ১২. কুকি সীমা ও বাস্তবিক নোট

* ব্রাউজার প্রতি কুকির সাইজ সাধারণত ~৪KB এবং প্রতি ডোমেইনে কুকির সংখ্যা সীমিত (সাধারণত ২০–৫০)। বড় ডেটা কুকিতে রাখা ঠিক নয়।
* `SameSite` ডিফল্ট আচরণ ব্রাউজারে পার্থক্য থাকতে পারে; তাই স্পষ্টভাবে সেট করুন।
* `sameSite: 'none'` হলে `secure: true` আবশ্যক। আধুনিক ব্রাউজারগুলো না দিলে ব্লক করবে।

# ১৩. অথেনটিকেশন চেক করার জন্য মিডলওয়্যার উদাহরণ

```js
function requireAuth(req, res, next) {
  const token = req.cookies.access_token; // cookie-parser থাকা দরকার
  if (!token) return res.status(401).json({ message: 'Not authenticated' });

  try {
    const payload = verifyJwt(token); // আপনার JWT ভেরিফাই ফাংশন
    req.user = payload;
    next();
  } catch (err) {
    return res.status(401).json({ message: 'Invalid token' });
  }
}
```

# ১৪. ছোট একটি পূর্ণাঙ্গ অ্যাপ উদাহরণ (ES module) — login কুকি সেট করে, protected route

```js
import express from 'express';
import cookieParser from 'cookie-parser';
import cors from 'cors';

const app = express();
app.use(express.json());
app.use(cookieParser('your-secret')); // যদি signed cookies ব্যবহার করতে চান

app.use(cors({
  origin: 'http://localhost:5173', // ফ্রন্টেন্ড origin
  credentials: true
}));

// fake user check
app.post('/login', (req, res) => {
  const { username } = req.body;
  // create token (JWT or session id)
  const token = createJwt({ username }); // আপনাকে implement করতে হবে
  res.cookie('access_token', token, {
    httpOnly: true,
    secure: false,       // প্রোডাকশনে true (HTTPS) রাখবেন
    maxAge: 1000 * 60 * 60, // ১ ঘন্টা
    sameSite: 'lax'
  });
  res.json({ ok: true });
});

app.get('/profile', (req, res) => {
  const token = req.cookies.access_token;
  if (!token) return res.status(401).send('Unauthorized');
  const user = verifyJwt(token);
  res.json({ user });
});

app.listen(3000);
```

# ১৫. প্রচলিত সমস্যা ও সমস্যা সমাধান (Troubleshooting)

* **ব্রাউজার কুকি পাঠাচ্ছেনা**: পরীক্ষা করুন `credentials`, `sameSite`, `secure` ও CORS সেটিংস ঠিক আছে কি না, এবং সার্ভার `Access-Control-Allow-Credentials: true` সেট করছে কি না।
* **কুকি সেট হলেও পরবর্তী রিকোয়েস্টে দেখা যাচ্ছে না**: `domain`/`path` মিলছে কি না, `secure` সেট করা কিন্তু আপনি HTTP-এ আছেন কি না, বা ব্রাউজার `SameSite` নীতির কারনে ব্লক করছে না তা দেখুন।
* **Signed cookie `req.cookies`-এ আসে, `req.signedCookies`-এ না আসে**: `res.cookie()` কলের সময়ে `{ signed: true }` সেট করেছেন কি না এবং `cookie-parser`-এ একই সিক্রেট দিয়েছেন কি না তা নিশ্চিত করুন।
* **কুকি ক্লিয়ার হচ্ছে না**: কুকি ক্লিয়ার করার সময় একই `path`/`domain` ব্যবহার করা হয়েছে কি না দেখুন।
* **কুকি আকস্মিকভাবে ওভাররাইট হচ্ছে**: `path` ও `domain` চেক করুন; একই নামের কুকি কিন্তু ভিন্ন path-এ থাকতে পারে — তা প্রভাব ফেলতে পারে।

# ১৬. কুকি বনামে localStorage — কখন কি ব্যবহার করবেন

* সংবেদনশীল টোকেনের জন্য `HttpOnly` কুকি ব্যবহার করুন (XSS থেকে রোধ)। তাই auth টোকেন কুকিতে রাখতে চান।
* `localStorage` বা `sessionStorage` সহজ, কিন্তু XSS দ্বারা সহজে পড়া/ছিনানো যায়; auth টোকেনের জন্য সাধারণত এটি সুপারিশ করা হয় না।

# ১৭. GDPR / কনসেন্ট নোট

যদি আপনি GDPR বা অনুরূপ আইনাধীন অঞ্চলে থাকেন, অপ্রয়োজনীয় কুকি (যেমন analytics, marketing) সেট করার আগে ব্যবহারকারীর সম্মতি নেওয়া লাগতে পারে। একটি কুকি কনসেন্ট ব্যানার দেখান এবং সম্মতি পাওয়া না পর্যন্ত সেই কুকি সেভ করবেন না।

# ১৮. ডেপ্লয়মেন্টের আগে দ্রুত চেকলিস্ট

* HTTPS চালু করুন এবং `secure: true` ব্যবহার করুন।
* auth কুকির জন্য `httpOnly` ব্যবহার করুন।
* উপযুক্ত `sameSite` সেট করুন।
* সংবেদনশীল টোকেনের মেয়াদ ছোট রাখুন।
* কুকিতে সর্বোচ্চ কম ডেটা রাখুন।
* ক্রস-ওরিজিন ফ্লো টেস্ট করুন (`credentials: 'include'`)।
* CSRF প্রতিরোধ ব্যবস্থা যোগ করুন।

---