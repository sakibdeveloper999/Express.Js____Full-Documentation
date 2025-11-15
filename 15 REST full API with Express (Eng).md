
---

# **RESTful API in Express.js**

---

## **1️⃣ RESTful API Principles — In Depth**

A RESTful API is more than just routes. Key **principles in practice**:

| Principle                     | Explanation & Implementation in Express                                                                                    |
| ----------------------------- | -------------------------------------------------------------------------------------------------------------------------- |
| **Stateless**                 | Each request contains all information. No session data on server. Use JWT for authentication.                              |
| **Client-Server Separation**  | Frontend (React/Vue) and backend (Express API) work independently. Backend exposes JSON endpoints; frontend consumes them. |
| **Resource-Based**            | Resources are nouns in URLs. Example: `/api/users` for all users, `/api/users/:id` for one user.                           |
| **Uniform Interface**         | Consistent API design — same structure, naming, response formats, status codes.                                            |
| **Cacheable**                 | Responses can be cached (HTTP `Cache-Control`) for performance.                                                            |
| **Layered System**            | API can sit behind proxies, gateways, or load balancers without client knowledge.                                          |
| **Code on Demand (optional)** | Rarely used. Allows sending scripts to clients dynamically.                                                                |

---

## **2️⃣ RESTful API Architecture in Express.js**

**Layered Architecture Example:**

```
Client (React/Vue/Angular)
        |
   [Routes/Controllers]
        |
   [Services/Business Logic]
        |
   [Models / Database Layer]
```

* **Routes:** Define endpoints.
* **Controllers:** Handle request/response logic.
* **Services:** Business logic, computations, external API calls.
* **Models:** Interface with the database.
* **Middleware:** Validation, authentication, logging, error handling.
* **Utils/Helpers:** Common reusable functions (e.g., hashing, formatting).

---

## **3️⃣ Advanced Route Organization**

**Grouping Routes with Router:**

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

* **Versioning API:** `/api/v1/users`, `/api/v2/users` for backward compatibility.
* **Modular routes** make maintenance easier in large projects.

---

## **4️⃣ CRUD Operations — Advanced Patterns**

### **Using Async/Await with Try/Catch Wrapper**

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

* This avoids repeating `try/catch` in every controller.

---

## **5️⃣ Advanced Query Features**

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

**Benefits:**

* Frontend can display pages.
* Large datasets don’t overload the server.
* Can filter or search by query parameters.

---

## **6️⃣ Error Handling Patterns**

Centralized error handling:

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

* Keeps all error responses consistent.
* Easy to maintain and extend.

---

## **7️⃣ Authentication & Authorization**

**JWT (JSON Web Token) Pattern:**

```javascript
import jwt from 'jsonwebtoken';

export const generateToken = (id) => {
  return jwt.sign({ id }, process.env.JWT_SECRET, { expiresIn: '30d' });
};
```

* **Protect Routes**:

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

* Role-based authorization is similar: check `req.user.role`.

---

## **8️⃣ Performance Optimization**

1. **Caching Responses**

   * Use Redis for frequently accessed data.
2. **Rate Limiting**

   * Prevent abuse with `express-rate-limit`.
3. **Compression**

   * Use `compression` middleware to reduce response size.
4. **Database Indexing**

   * Add indexes to frequently queried fields for speed.

---

## **9️⃣ Security Best Practices**

1. **HTTP Headers**

   * `helmet` middleware for secure headers.
2. **Input Validation**

   * Prevent SQL/NoSQL injections with `express-validator` or `Joi`.
3. **HTTPS**

   * Encrypt traffic with SSL/TLS.
4. **Authentication**

   * Use JWT, OAuth2, or API keys.
5. **CORS**

   * Configure with `cors` middleware to allow only trusted domains.
6. **Logging & Monitoring**

   * `morgan`, `winston` for logs.
   * Monitor performance and errors in production.

---

## **🔟 Testing RESTful API**

1. **Unit Testing Controllers**

   * Using `Jest` and `supertest`:

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

2. **Integration Testing**

   * Test API endpoints with a real test database.
3. **Postman / Insomnia**

   * Useful for manual testing and generating collections.

---

## **1️⃣1️⃣ Real-World Use Cases**

1. **E-commerce Backend**

   * Products, categories, orders, payment endpoints.
2. **Social Network API**

   * Users, posts, likes, comments, friends.
3. **Blog Platform**

   * CRUD posts, authors, tags.
4. **Microservices**

   * RESTful API to communicate between independent services.
5. **Mobile Apps**

   * React Native or Flutter apps consuming your API.

---

## **1️⃣2️⃣ RESTful API Best Practices**

| Principle                  | Implementation Example                   |
| -------------------------- | ---------------------------------------- |
| Use nouns, not verbs       | `/api/users` instead of `/api/getUsers`  |
| Version your API           | `/api/v1/users`                          |
| Proper HTTP status codes   | `200 OK`, `201 Created`, `404 Not Found` |
| Consistent JSON response   | `{ success: true, data: {...} }`         |
| Secure endpoints           | JWT auth, CORS, HTTPS                    |
| Paginate large results     | `/api/users?page=1&limit=10`             |
| Centralized error handling | `errorHandler` middleware                |

---

## ✅ **Next Level: Microservices & REST API**

* Split your API into **microservices**:

  * User Service, Product Service, Order Service.
* Use **RESTful APIs or gRPC** for communication.
* Implement **API Gateway** for routing, authentication, logging.

---
