
# **Express.js-এ Authentication: গভীর বিশ্লেষণ**

Authentication শুধু পাসওয়ার্ড যাচাই করা নয়—এটি **ইউজার আইডেন্টিটি নিরাপদভাবে ম্যানেজ করা**, **সেশন, টোকেন, রোলস, এবং পারমিশন** নিয়ন্ত্রণ করার প্রক্রিয়া।

---

## **1️⃣ Authentication বনাম Authorization**

| কনসেপ্ট            | সংজ্ঞা                          | Express.js উদাহরণ                                      |
| ------------------ | ------------------------------- | ------------------------------------------------------ |
| **Authentication** | ইউজার কে তা যাচাই করা           | ইমেইল & পাসওয়ার্ড দিয়ে লগইন, OAuth লগইন              |
| **Authorization**  | ইউজার কী করতে পারবে তা নির্ধারণ | ইউজার কি 'admin' রোল পেয়েছে `/admin` অ্যাক্সেসের জন্য |

> **মনে রাখবেন:** Authentication সর্বদা Authorization-এর আগে হয়।

---

## **2️⃣ Authentication আর্কিটেকচার**

### **a) Session-based Authentication (Stateful)**

* সার্ভার লগইন করা ইউজারদের ট্র্যাক রাখে।
* **সার্ভার-সাইড স্টোরেজ** ব্যবহার করে: memory, database, Redis।
* Workflow:

1. ইউজার লগইন করে।
2. সার্ভার একটি **session ID** তৈরি করে এবং স্টোরেজে রাখে।
3. ক্লায়েন্টকে **cookie** পাঠায় session ID সহ।
4. ক্লায়েন্ট প্রতিটি request-এ cookie পাঠায়।
5. সার্ভার session ID যাচাই করে → access দেয়।

**Pros:**

* সহজে ইমপ্লিমেন্ট করা যায়।
* Sessions যেকোন সময় invalidate করা যায়।

**Cons:**

* API বা microservices-এ স্কেল করা কঠিন যদি session storage share না হয়।

**Express প্যাকেজ উদাহরণ:** `express-session`, `connect-mongo`, `connect-redis`

---

### **b) Token-based Authentication (Stateless)**

* সার্ভার session রাখে না; সব তথ্য **টোকেনে এনকোড করা থাকে** (সাধারণত JWT)।
* Workflow:

1. ইউজার লগইন করে।
2. সার্ভার **JWT** তৈরি করে এবং সিক্রেট/প্রাইভেট কী দিয়ে sign করে।
3. ক্লায়েন্ট টোকেন রাখে (cookie বা localStorage)।
4. ক্লায়েন্ট প্রতিটি request-এ **Authorization header** পাঠায়।
5. সার্ভার টোকেন যাচাই করে → access দেয়।

**Pros:**

* Stateless → সহজে স্কেল করা যায়।
* SPA বা মোবাইল অ্যাপের জন্য উপযুক্ত।

**Cons:**

* JWT সাধারণত expire না হওয়া পর্যন্ত invalidate করা যায় না।
* Token blacklist ব্যবহার করতে হবে যদি revoke করতে চাই।

---

### **c) OAuth / Social Login**

* ইউজারের authentication delegated হয় providers: Google, Facebook, GitHub।
* ব্যবহার হয় **OAuth2 টোকেন**।
* Express middleware উদাহরণ: `passport-google-oauth20`।

**Workflow:**

1. ইউজার “Login with Google” ক্লিক করে।
2. ইউজার Google login page এ redirect হয়।
3. Google redirect করে সার্ভারে **code** নিয়ে।
4. সার্ভার code exchange করে access token → optionally local user তৈরি করে।

---

## **3️⃣ Password Handling**

**কখনো পাসওয়ার্ড plain text এ সংরক্ষণ করবেন না।**

Best practices:

1. **Hash passwords**: bcrypt, scrypt, argon2 ব্যবহার করুন।
2. **Salt** ব্যবহার করুন rainbow table attack থেকে রক্ষা পেতে।
3. **Pepper** ব্যবহার করুন: server secret string পাসওয়ার্ডের সাথে অ্যাপেন্ড করে হ্যাশ করা।

```js
import bcrypt from "bcryptjs";

const hashPassword = async (password) => {
  const salt = await bcrypt.genSalt(12); // 12 rounds
  return await bcrypt.hash(password, salt);
};
```

---

## **4️⃣ JWT বিশদ**

JWT = **JSON Web Token**, একটি Stateless authentication পদ্ধতি।

**Structure:**

```
Header.Payload.Signature
```

* **Header**: `{ "alg": "HS256", "typ": "JWT" }`
* **Payload**: `{ "id": "userId", "role": "admin" }`
* **Signature**: HMACSHA256(base64UrlEncode(header) + "." + base64UrlEncode(payload), secret)

**Advantages:**

* Stateless → সহজে স্কেল করা যায়।
* Payload-এ রোলস এবং permissions রাখা যায়।
* API, SPA, microservices-এ ব্যবহারযোগ্য।

**Disadvantages:**

* Expire না হওয়া পর্যন্ত revoke করা যায় না।
* Secret leak হলে সব token compromised।
* সবসময় **HTTPS** ব্যবহার করতে হবে।

**Best Practices:**

1. Short-lived tokens (5–15 min)
2. Refresh tokens ব্যবহার করুন long sessions এর জন্য
3. **HTTP-only cookie**-তে token রাখুন XSS থেকে রক্ষা করতে
4. প্রতিটি request-এ signature validate করুন

---

## **5️⃣ রুট প্রোটেকশন**

**Middleware প্যাটার্ন:**

```js
import jwt from "jsonwebtoken";

export const authenticate = (roles = []) => (req, res, next) => {
  const token = req.cookies.token || req.header("Authorization")?.replace("Bearer ", "");

  if (!token) return res.status(401).json({ message: "No token provided" });

  try {
    const decoded = jwt.verify(token, process.env.JWT_SECRET);
    req.user = decoded;

    if (roles.length && !roles.includes(decoded.role))
      return res.status(403).json({ message: "Forbidden" });

    next();
  } catch (err) {
    return res.status(401).json({ message: "Invalid token" });
  }
};
```

**ব্যাখ্যা:**

* `roles` parameter দিয়ে authorization করা যায়।
* Middleware token existence, validity এবং role যাচাই করে।

---

## **6️⃣ Refresh Tokens (Advanced JWT Handling)**

সমস্যা: JWT খুব দ্রুত expire হয় → ইউজারকে বারবার লগইন করতে হয়।
সমাধান: **Refresh tokens**

1. Access token → short-lived (5–15 min)
2. Refresh token → long-lived (7–30 days)

**Workflow:**

* Access token expire হলে → client refresh token পাঠায় → সার্ভার validate করে → নতুন access token ইস্যু করে।
* Refresh token নিরাপদে সংরক্ষণ করুন (HTTP-only cookie বা DB)

```js
// /token route
router.post("/token", async (req, res) => {
  const refreshToken = req.cookies.refreshToken;
  if (!refreshToken) return res.status(401).json({ message: "No refresh token" });

  try {
    const decoded = jwt.verify(refreshToken, process.env.REFRESH_SECRET);
    const newAccessToken = jwt.sign({ id: decoded.id }, process.env.JWT_SECRET, { expiresIn: "15m" });
    res.cookie("token", newAccessToken, { httpOnly: true });
    res.json({ accessToken: newAccessToken });
  } catch (err) {
    res.status(401).json({ message: "Invalid refresh token" });
  }
});
```

---

## **7️⃣ Multi-Factor Authentication (MFA)**

* অতিরিক্ত security layer।
* উদাহরণ: login করার পরে OTP verification।
* প্যাকেজ: `speakeasy` (TOTP), `nodemailer` (email), `twilio` (SMS)

**Workflow:**

1. পাসওয়ার্ড দিয়ে login
2. সার্ভার TOTP generate করে পাঠায় ইউজারকে
3. ইউজার OTP submit করে
4. সার্ভার verify করে → session/token ইস্যু করে

---

## **8️⃣ Security Considerations**

1. সবসময় **password hash** করুন, plain text রাখবেন না।
2. **HTTPS** ব্যবহার করুন MITM attack থেকে রক্ষা পেতে।
3. **HTTP-only cookies** → XSS থেকে JWT রক্ষা।
4. **CSRF protection** → cookie ভিত্তিক auth এর জন্য।
5. Login endpoints rate-limit করুন → brute force prevent করতে।
6. Error messages কম তথ্যযুক্ত রাখুন → attackers ইউজার ইমেইল না জানতে পারে।
7. Multiple failed attempts → account lock বা CAPTCHA।
8. **helmet.js** ব্যবহার করুন Express-এ secure headers সেট করতে।

---

## **9️⃣ Use Cases / Advanced Scenarios**

| Scenario            | Recommended Approach                  |
| ------------------- | ------------------------------------- |
| Traditional web app | Session-based + cookie                |
| SPA (React, Vue)    | JWT + HTTP-only cookie                |
| API for mobile app  | JWT + refresh token                   |
| Multi-role app      | JWT + roles + middleware              |
| Social login        | OAuth2 + JWT/session                  |
| High-security app   | MFA + short-lived tokens + monitoring |

---

## **10️⃣ Example Project Architecture**

```
/project
  /models
    User.js
  /routes
    auth.js
    user.js
  /middleware
    auth.js
  server.js
  config.js
```

* `User.js` → user schema, password hashing
* `auth.js` → login, register, refresh token, logout
* `user.js` → protected routes (dashboard, profile)
* `middleware/auth.js` → verify token + roles
* `server.js` → main Express app setup

---

## ✅ **সারসংক্ষেপ**

Express.js Authentication **multi-layered**:

1. **User identity verification** → password/email, OAuth, token
2. **Session বা token storage** → stateful vs stateless
3. **Route protection** → middleware identity & role যাচাই
4. **Advanced features** → refresh tokens, MFA, rate limiting
5. **Security best practices** → password hashing, HTTPS, CSRF, XSS prevention

> এই গভীর বিশ্লেষণের মাধ্যমে আপনি জানেন **Authentication কিভাবে কাজ করে**, **Express.js-এ এটি নিরাপদভাবে কিভাবে ইমপ্লিমেন্ট করবেন**, এবং **রিয়েল-ওয়ার্ল্ড ইউজকেসগুলো কেমন হবে**।

---
