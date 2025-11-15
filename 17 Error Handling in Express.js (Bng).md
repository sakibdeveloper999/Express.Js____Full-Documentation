
---

# **ডিপ ডাইভ: Express.js-এ এরর হ্যান্ডলিং**

---

## **1️⃣ Express.js-এ এরর হ্যান্ডলিং কী?**

Express.js-এ **এরর হ্যান্ডলিং** হলো **রিকোয়েস্ট-রেসপন্স চক্রে যে কোনো এরর ঘটলে তা ধরার, পরিচালনা করার এবং ক্লায়েন্টকে সঠিকভাবে রেসপন্স দেওয়ার প্রক্রিয়া।**

ঠিকমতো এরর হ্যান্ডলিং না করলে:

* অ্যাপ **ক্র্যাশ করতে পারে**
* ক্লায়েন্টকে **অসঙ্গত বা ব্যবহারকারী-বান্ধব নয় এমন রেসপন্স** যেতে পারে

Express.js-এ **এরর এর মূল তিনটি উৎস**:

1. **Synchronous errors (সিঙ্ক্রোনাস এরর)** – কোড এক্সিকিউশনের সময় তৎক্ষণাৎ ঘটে।
2. **Asynchronous errors (অ্যাসিঙ্ক্রোনাস এরর)** – কলব্যাক, প্রমিস, async/await-এ ঘটে।
3. **Operational errors (অপারেশনাল এরর)** – ভুল ইনপুট, resource না পাওয়া, বা external API এরর।

---

## **2️⃣ Express.js-এ এরর হ্যান্ডলিং মিডলওয়্যার**

Express-এ এরর হ্যান্ডলিং এর জন্য **স্পেশাল মিডলওয়্যার** ব্যবহার করা হয়। এটি **চারটি প্যারামিটার** নিয়ে তৈরি হয়:

```js
function errorHandler(err, req, res, next) {
    console.error(err.stack);
    res.status(500).json({ message: err.message });
}
```

**মূল পয়েন্টসমূহ:**

* `err` – এরর অবজেক্ট।
* `req` – রিকোয়েস্ট অবজেক্ট।
* `res` – রেসপন্স অবজেক্ট।
* `next` – পরবর্তী মিডলওয়্যার।

> চারটি প্যারামিটার না দিলে Express এটি **এরর হ্যান্ডলার হিসেবে চিনবে না**।

---

## **3️⃣ Express-এ এরর কিভাবে propagate হয়**

1. **Synchronous route errors**

```js
app.get('/sync', (req, res) => {
    throw new Error("সিঙ্ক্রোনাস এরর!");
});
```

* Express স্বয়ংক্রিয়ভাবে এররটি **এরর হ্যান্ডলার মিডলওয়্যারে পাঠায়।**

2. **Asynchronous route errors (Promise)**

```js
app.get('/async', async (req, res, next) => {
    try {
        await someAsyncFunction();
        res.send("সফল!");
    } catch (err) {
        next(err); // এরর মিডলওয়্যারে পাঠানো
    }
});
```

* `async/await` এর এরর **স্বয়ংক্রিয়ভাবে propagation হয় না**, `next(err)` কল করতে হবে।

3. **Next ফাংশন দিয়ে এরর পাঠানো**

```js
app.get('/next', (req, res, next) => {
    const err = new Error("ম্যানুয়াল এরর");
    next(err);
});
```

* `next(err)` এর মাধ্যমে সরাসরি **এরর মিডলওয়্যারে পাঠানো যায়।**

---

## **4️⃣ এরর হ্যান্ডলিং এর বেস্ট প্র্যাকটিস**

### **A. সেন্ট্রালাইজড এরর হ্যান্ডলিং**

প্রতিটি রুটে এরর হ্যান্ডল করার পরিবর্তে, **একটি গ্লোবাল এরর হ্যান্ডলার ব্যবহার করুন**:

```js
// app.js বা server.js
app.use((err, req, res, next) => {
    console.error(err.stack);
    const status = err.status || 500;
    res.status(status).json({
        success: false,
        error: err.message || "Internal Server Error",
    });
});
```

* গ্লোবাল হ্যান্ডলিং অ্যাপকে **মেইনটেইন এবং ডিবাগ করা সহজ করে।**

---

### **B. কাস্টম এরর ক্লাস**

Better organization এর জন্য **কাস্টম এরর ক্লাস** তৈরি করুন:

```js
class AppError extends Error {
    constructor(message, statusCode) {
        super(message);
        this.statusCode = statusCode || 500;
        this.isOperational = true; // known errors চিহ্নিত করতে
    }
}

// ব্যবহার:
app.get('/notfound', (req, res, next) => {
    next(new AppError("Resource পাওয়া যায়নি", 404));
});
```

* **Operational errors** এবং **programming errors** আলাদা করার সুবিধা।

---

### **C. Async Wrapper Utility**

Async route গুলোর জন্য `try/catch` কমাতে wrapper ব্যবহার করুন:

```js
const asyncHandler = fn => (req, res, next) => {
    Promise.resolve(fn(req, res, next)).catch(next);
};

// ব্যবহার
app.get('/users', asyncHandler(async (req, res) => {
    const users = await User.find();
    res.json(users);
}));
```

* Async route এর এরর **সহজে ফরওয়ার্ড করা যায়।**

---

### **D. Validation এবং Known Errors**

Validation লাইব্রেরি (Joi, express-validator) ব্যবহার করে ইনপুট যাচাই করুন:

```js
if (!req.body.name) {
    return next(new AppError("নাম আবশ্যক", 400));
}
```

* নিশ্চিত করে যে ক্লায়েন্ট **ঠিকঠাক মেসেজ** পায় এবং HTTP স্ট্যাটাস ঠিক থাকে।

---

## **5️⃣ Express-এ এরর হ্যান্ডলিং ফ্লো**

1. ক্লায়েন্ট রিকোয়েস্ট পাঠায় → Route handler চলে।
2. Route handler এরর ডিটেক্ট করে:

   * Synchronous এরর throw করে।
   * Async এরর হলে `next(err)` ব্যবহার করে।
3. Express **এরর মিডলওয়্যার** (৪ প্যারামিটার) খুঁজে বের করে।
4. এরর মিডলওয়্যার লগ করে এবং **JSON বা HTML** রেসপন্স দেয়।
5. ক্লায়েন্ট structured error message পায়।

**Flow Diagram (সরাসরি):**

```
[Request] --> [Route Handler] --> [Error?] --> Yes --> [Error Middleware] --> [Response]
```

---

## **6️⃣ রিয়েল-ওয়ার্ল্ড ইউস কেস**

1. **API Responses**

```js
res.status(404).json({ success: false, message: "User পাওয়া যায়নি" });
```

2. **ডাটাবেস এরর**

* DB এরর try/catch-এ ধরুন এবং middleware-এ পাঠান।

3. **Authentication / Authorization Errors**

```js
next(new AppError("Unauthorized access", 401));
```

4. **404 Not Found**

```js
app.use((req, res, next) => {
    next(new AppError("Route পাওয়া যায়নি", 404));
});
```

5. **Logging & Monitoring**

* Winston বা Morgan দিয়ে centralized logging করুন।

---

## **7️⃣ অ্যাডভান্সড টিপস**

* **Development vs Production আলাদা করা**

```js
if (process.env.NODE_ENV === 'development') {
    console.error(err.stack);
} else {
    // production-এ stack trace দেখাবেন না
}
```

* **Third-party Error Reporting**

  * Sentry, Rollbar বা LogRocket ব্যবহার করে real-time monitoring।

* **Sensitive info লুকানো**

* **HTTP Status code সঠিক ব্যবহার**: 400, 401, 403, 404, 500 ইত্যাদি।

---

## **8️⃣ সম্পূর্ণ উদাহরণ**

```js
const express = require('express');
const app = express();

class AppError extends Error {
    constructor(message, statusCode) {
        super(message);
        this.statusCode = statusCode || 500;
        this.isOperational = true;
    }
}

// Async wrapper
const asyncHandler = fn => (req, res, next) => {
    Promise.resolve(fn(req, res, next)).catch(next);
};

// Routes
app.get('/sync', (req, res) => {
    throw new Error("সিঙ্ক্রোনাস এরর!");
});

app.get('/async', asyncHandler(async (req, res, next) => {
    throw new AppError("Async এরর হয়েছে", 502);
}));

app.get('/notfound', (req, res, next) => {
    next(new AppError("Resource পাওয়া যায়নি", 404));
}));

// 404 handler
app.use((req, res, next) => {
    next(new AppError("Route পাওয়া যায়নি", 404));
}));

// Centralized Error Middleware
app.use((err, req, res, next) => {
    console.error(err.stack);
    const status = err.statusCode || 500;
    res.status(status).json({
        success: false,
        message: err.message || "Internal Server Error",
    });
});

app.listen(3000, () => console.log("Server 3000 পোর্টে চলছে"));
```

---

✅ **সারসংক্ষেপ**

* Express এরর হ্যান্ডলিং **middleware-based**।
* সবসময় **centralized error middleware** ব্যবহার করুন।
* Async routes এরর **auto-forward করতে wrapper** ব্যবহার করুন।
* **Custom error classes** ব্যবহার করুন।
* ক্লায়েন্টকে **user-friendly responses** দিন।
* **ডিবাগ এবং লগিং** নিশ্চিত করুন।
* Development এবং Production আলাদা করুন।
* **404 এবং unknown routes** gracefully handle করুন।

---