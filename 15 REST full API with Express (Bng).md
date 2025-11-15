
---
# **এক্সপ্রেসে RESTful API-এর সম্পূর্ণ বিশ্লেষণ**

---

## **1️⃣ RESTful API নীতিমালা — বিস্তারিতভাবে**

RESTful API শুধু রুটের উপর নির্ভর করে না। মূল **নীতিসমূহ ও বাস্তবায়ন**:

| নীতি                                   | ব্যাখ্যা ও এক্সপ্রেসে বাস্তবায়ন                                                                                                          |
| -------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------- |
| **Stateless (স্টেটলেস)**               | প্রতিটি রিকোয়েস্টে সব তথ্য থাকতে হবে। সার্ভার কোন ক্লায়েন্ট স্টেট সংরক্ষণ করবে না। Authentication-এ JWT ব্যবহার করুন।                   |
| **Client-Server Separation**           | ফ্রন্টএন্ড (React/Vue) এবং ব্যাকএন্ড (Express API) স্বাধীনভাবে কাজ করে। ব্যাকএন্ড JSON এন্ডপয়েন্ট প্রদান করে, ফ্রন্টএন্ড তা ব্যবহার করে। |
| **Resource-Based (রিসোর্স ভিত্তিক)**   | URL-এ সবকিছু noun (সত্তা) হিসাবে প্রকাশিত। উদাহরণ: `/api/users` সকল ইউজারের জন্য, `/api/users/:id` একটি নির্দিষ্ট ইউজারের জন্য।           |
| **Uniform Interface (সদৃশ ইন্টারফেস)** | API ডিজাইন সবসময় একরকম হবে — স্ট্রাকচার, নামকরণ, রেসপন্স ফরম্যাট, স্ট্যাটাস কোড।                                                         |
| **Cacheable (ক্যাশেবল)**               | রেসপন্স ক্যাশ করা যায় (HTTP `Cache-Control`) পারফরম্যান্স উন্নত করার জন্য।                                                               |
| **Layered System (লেয়ার্ড সিস্টেম)**  | API প্রক্সি, গেটওয়ে বা লোড ব্যালান্সারের পিছনে থাকতে পারে ক্লায়েন্ট জানার প্রয়োজন নেই।                                                 |
| **Code on Demand (ঐচ্ছিক)**            | রেয়ারলি ব্যবহৃত হয়। ক্লায়েন্টকে ডাইনামিক স্ক্রিপ্ট পাঠানোর সুযোগ দেয়।                                                                 |

---

## **2️⃣ RESTful API আর্কিটেকচার এক্সপ্রেসে**

**লেয়ার্ড আর্কিটেকচার উদাহরণ:**

```
Client (React/Vue/Angular)
        |
   [Routes/Controllers]
        |
   [Services/Business Logic]
        |
   [Models / Database Layer]
```

* **Routes:** এন্ডপয়েন্ট নির্ধারণ।
* **Controllers:** রিকোয়েস্ট/রেসপন্স লজিক।
* **Services:** ব্যবসায়িক লজিক, হিসাব, বাইরের API কল।
* **Models:** ডাটাবেস ইন্টারফেস।
* **Middleware:** ভ্যালিডেশন, অটেনটিকেশন, লগিং, এরর হ্যান্ডলিং।
* **Utils/Helpers:** রিইউজেবল ফাংশন (যেমন হ্যাশিং, ফরম্যাটিং)।

---

## **3️⃣ অ্যাডভান্সড রুট অর্গানাইজেশন**

**Router দিয়ে গ্রুপ করা:**

```javascript
// routes/api.js
import express from 'express';
import userRoutes from './userRoutes.js';
import postRoutes from './postRoutes.js';

const router = express.Router();

router.use('/users', userRoutes);
router.use('/posts', postRoutes);

export default router;
```

* **API Versioning:** `/api/v1/users`, `/api/v2/users`
* বড় প্রজেক্টে **মডুলার রুট** রক্ষণাবেক্ষণ সহজ করে।

---

## **4️⃣ CRUD অপারেশন — অ্যাডভান্সড প্যাটার্ন**

### **Async/Await & Try/Catch Wrapper**

```javascript
// utils/asyncHandler.js
export const asyncHandler = fn => (req, res, next) => {
  Promise.resolve(fn(req, res, next)).catch(next);
};
```

```javascript
// controllers/userController.js
import User from '../models/userModel.js';
import { asyncHandler } from '../utils/asyncHandler.js';

export const getUsers = asyncHandler(async (req, res) => {
  const users = await User.find();
  res.status(200).json(users);
});
```

* প্রতিটি কন্ট্রোলারে `try/catch` লিখতে হয় না।

---

## **5️⃣ অ্যাডভান্সড কোয়েরি ফিচার**

### **Pagination, Filtering, Sorting, Searching**

```javascript
export const getUsers = asyncHandler(async (req, res) => {
  const page = Number(req.query.page) || 1;
  const limit = Number(req.query.limit) || 10;
  const keyword = req.query.keyword
    ? { name: { $regex: req.query.keyword, $options: 'i' } }
    : {};

  const users = await User.find({ ...keyword })
    .limit(limit)
    .skip(limit * (page - 1));

  const count = await User.countDocuments({ ...keyword });

  res.json({ users, page, pages: Math.ceil(count / limit) });
});
```

**উপকারিতা:**

* ফ্রন্টএন্ডে পেজ ডিসপ্লে করা যায়।
* বড় ডাটাসেট সার্ভারকে ওভারলোড করে না।
* কোয়েরি প্যারামিটার দিয়ে সার্চ/ফিল্টার করা যায়।

---

## **6️⃣ এরর হ্যান্ডলিং প্যাটার্ন**

```javascript
// middleware/errorMiddleware.js
export const notFound = (req, res, next) => {
  const error = new Error(`Not Found - ${req.originalUrl}`);
  res.status(404);
  next(error);
};

export const errorHandler = (err, req, res, next) => {
  const statusCode = res.statusCode === 200 ? 500 : res.statusCode;
  res.status(statusCode).json({
    message: err.message,
    stack: process.env.NODE_ENV === 'production' ? null : err.stack,
  });
};
```

* সব এরর কনসিসটেন্ট থাকে।
* রক্ষণাবেক্ষণ সহজ।

---

## **7️⃣ Authentication & Authorization**

**JWT প্যাটার্ন:**

```javascript
import jwt from 'jsonwebtoken';

export const generateToken = (id) => {
  return jwt.sign({ id }, process.env.JWT_SECRET, { expiresIn: '30d' });
};
```

**Protect Routes:**

```javascript
import jwt from 'jsonwebtoken';
import User from '../models/userModel.js';

export const protect = async (req, res, next) => {
  let token;
  if (req.headers.authorization?.startsWith('Bearer')) {
    token = req.headers.authorization.split(' ')[1];
    try {
      const decoded = jwt.verify(token, process.env.JWT_SECRET);
      req.user = await User.findById(decoded.id).select('-password');
      next();
    } catch (err) {
      res.status(401).json({ message: 'Not authorized' });
    }
  }
  if (!token) res.status(401).json({ message: 'No token' });
};
```

* Role-based authorization: `req.user.role` চেক করুন।

---

## **8️⃣ পারফরম্যান্স অপ্টিমাইজেশন**

1. **Response caching:** Redis ব্যবহার করুন।
2. **Rate limiting:** `express-rate-limit` দিয়ে অব্যবহৃত রিকোয়েস্ট ব্লক করুন।
3. **Compression:** `compression` middleware ব্যবহার করুন।
4. **Database Indexing:** প্রায়ই কোয়েরি করা ফিল্ডে ইনডেক্স যোগ করুন।

---

## **9️⃣ সিকিউরিটি বেস্ট প্র্যাকটিস**

1. **HTTP Headers:** `helmet` middleware ব্যবহার।
2. **Input Validation:** `express-validator` বা `Joi` ব্যবহার করে SQL/NoSQL Injection প্রতিরোধ।
3. **HTTPS:** ট্রাফিক এনক্রিপশন।
4. **Authentication:** JWT, OAuth2, API Keys।
5. **CORS:** শুধু ট্রাস্টেড ডোমেইনের জন্য।
6. **Logging & Monitoring:** `morgan`, `winston`। প্রোডাকশনে পারফরম্যান্স মনিটরিং।

---

## **🔟 RESTful API Testing**

1. **Unit Testing Controllers:** `Jest` ও `supertest` ব্যবহার করুন।

```javascript
import request from 'supertest';
import app from '../server.js';

describe('GET /api/users', () => {
  it('should return users', async () => {
    const res = await request(app).get('/api/users');
    expect(res.statusCode).toEqual(200);
    expect(Array.isArray(res.body)).toBeTruthy();
  });
});
```

2. **Integration Testing:** টেস্ট ডাটাবেস দিয়ে API পরীক্ষা।
3. **Postman/Insomnia:** ম্যানুয়াল টেস্টিং ও কালেকশন তৈরির জন্য।

---

## **1️⃣1️⃣ বাস্তব জীবনের ইউজ কেস**

1. **E-commerce Backend:** প্রোডাক্ট, ক্যাটাগরি, অর্ডার, পেমেন্ট।
2. **Social Network API:** ইউজার, পোস্ট, লাইক, কমেন্ট, ফ্রেন্ড।
3. **Blog Platform:** CRUD পোস্ট, অথর, ট্যাগ।
4. **Microservices:** স্বতন্ত্র সার্ভিসের মধ্যে কমিউনিকেশন।
5. **Mobile Apps:** React Native বা Flutter API ব্যবহার।

---

## **1️⃣2️⃣ RESTful API Best Practices**

| নীতি                       | উদাহরণ                                   |
| -------------------------- | ---------------------------------------- |
| Use nouns, not verbs       | `/api/users` ✅, `/api/getUsers` ❌        |
| Version your API           | `/api/v1/users`                          |
| Proper HTTP status codes   | `200 OK`, `201 Created`, `404 Not Found` |
| Consistent JSON response   | `{ success: true, data: {...} }`         |
| Secure endpoints           | JWT auth, CORS, HTTPS                    |
| Paginate large results     | `/api/users?page=1&limit=10`             |
| Centralized error handling | `errorHandler` middleware                |

---

## ✅ **পরবর্তী ধাপ: Microservices & REST API**

* API-কে **microservices**-এ ভাগ করুন: User Service, Product Service, Order Service।
* সার্ভিস কমিউনিকেশনের জন্য **RESTful API বা gRPC** ব্যবহার।
* **API Gateway** ব্যবহার করুন: রাউটিং, অটেনটিকেশন, লগিং।

---
