
# 1. Quick concepts — what is a cookie?

A cookie is a small piece of data the server asks the browser to store and send back on later requests. Cookies are set by the server using the `Set-Cookie` header and read by the server from the `Cookie` header on subsequent requests.

# 2. Install what you’ll usually need

The common helper library is `cookie-parser` which parses incoming cookies and supports signed cookies.

```bash
npm install express cookie-parser
# or with yarn
yarn add express cookie-parser
```

# 3. Minimal Express (ES modules) setup

```js
// index.mjs or server.mjs (ES module)
import express from 'express';
import cookieParser from 'cookie-parser';

const app = express();
const PORT = 3000;

// If you want signed cookies, pass a secret; otherwise omit.
app.use(cookieParser('my-secret-string'));

app.use(express.json());

app.get('/', (req, res) => {
  res.send('Hello');
});

app.listen(PORT, () => console.log(`Listening ${PORT}`));
```

# 4. Setting cookies from server

Use `res.cookie(name, value, options)`.

```js
app.get('/set-cookie', (req, res) => {
  // Basic cookie
  res.cookie('theme', 'dark', { maxAge: 1000 * 60 * 60 * 24 }); // 1 day

  // HttpOnly cookie (not accessible to client-side JS) - good for auth tokens
  res.cookie('sessionId', 'abc123', {
    httpOnly: true,
    secure: process.env.NODE_ENV === 'production', // only over HTTPS in prod
    maxAge: 1000 * 60 * 60 * 24 * 7, // 7 days
    sameSite: 'lax'
  });

  // Signed cookie
  res.cookie('signedName', 'sakib', { signed: true, httpOnly: true });

  res.send('Cookies set');
});
```

**Important cookie options**

* `httpOnly: true` — not accessible via `document.cookie` (prevents some XSS theft).
* `secure: true` — only sent over HTTPS.
* `maxAge` (ms) or `expires` (Date) — lifetime.
* `sameSite: 'lax' | 'strict' | 'none'` — controls cross-site sending. `none` requires `secure: true`.
* `path` and `domain` — restrict scope.
* `signed: true` — if you used `cookie-parser` with a secret. Signed cookies are tamper-detectable.

# 5. Reading cookies in request handlers

`cookie-parser` populates `req.cookies` (and `req.signedCookies` for signed cookies).

```js
app.get('/read', (req, res) => {
  console.log(req.cookies);            // all unsigned cookies
  console.log(req.signedCookies);      // all signed cookies
  const theme = req.cookies.theme;
  res.json({ theme });
});
```

# 6. Clearing/deleting cookies

Use `res.clearCookie(name, options?)`. Match `path`/`domain` if used.

```js
app.get('/logout', (req, res) => {
  res.clearCookie('sessionId'); // remove cookie
  res.send('Logged out');
});
```

# 7. Signed cookies — example

```js
// cookieParser was initialized with 'my-secret-string'
app.get('/set-signed', (req, res) => {
  res.cookie('user', 'mdsakib', { signed: true, httpOnly: true });
  res.send('Signed cookie set');
});
app.get('/check-signed', (req, res) => {
  res.send({ signed: req.signedCookies.user || null });
});
```

# 8. Cookies for authentication (common patterns)

* **Session tokens (server-side):** store a session id in an `HttpOnly` cookie. Server keeps session data in memory/Redis/DB keyed by id (using `express-session` or manual).
* **JWT in cookie:** store access or refresh tokens in `HttpOnly, Secure` cookies. Keep access token short-lived and refresh in HttpOnly cookie.
* **Double-submit cookie for CSRF:** set a cookie and require the client to send the same value in a custom header — server validates match.

Example: set auth cookie (HttpOnly + Secure):

```js
res.cookie('access_token', token, {
  httpOnly: true,
  secure: true,       // must be HTTPS in production
  sameSite: 'lax',
  maxAge: 1000 * 60 * 15 // 15 minutes
});
```

# 9. CORS and cross-site cookies (important)

If your frontend runs on a different origin, the browser will only send cookies if:

* client fetch uses `credentials: 'include'` (or `credentials: 'same-origin'` for same origin),
* server sets `Access-Control-Allow-Credentials: true`,
* server's CORS `origin` is not `*` (must be the actual origin),
* cookies that are cross-site typically need `sameSite: 'none'` **and** `secure: true`.

Example server config with `cors`:

```js
import cors from 'cors';

app.use(cors({
  origin: 'https://app.example.com',
  credentials: true,
}));

// Then client fetch:
fetch('https://api.example.com/data', {
  method: 'GET',
  credentials: 'include'
});
```

# 10. Setting cookie from client (JavaScript)

* To set via `document.cookie` (not recommended for auth cookies because they can be read by JS):

```js
document.cookie = "name=val;path=/;max-age=3600";
```

* When using `fetch` to send or receive cookies cross-site:

```js
fetch('https://api.example.com/login', {
  method: 'POST',
  credentials: 'include',
  body: JSON.stringify({ username, password }),
  headers: { 'Content-Type': 'application/json' }
});
```

# 11. Security best practices

* Use `HttpOnly` for tokens you don’t want JS to access.
* Use `Secure` in production (HTTPS required).
* Use `SameSite=Lax` or `Strict` to reduce CSRF risk; `None` only when necessary (e.g., third-party cross-site).
* Prefer storing minimal data in cookies (no big JSON blobs).
* Keep cookie payloads small (< ~4096 bytes) and avoid sensitive data in plaintext.
* Use signed cookies or HMACs if you store important data client-side.
* Prefer server-side sessions (session id in cookie) for sensitive server state.
* Implement CSRF protection: `sameSite`, CSRF tokens, or double-submit cookie.
* When using JWT in cookie: store refresh token in HttpOnly cookie; rotate refresh tokens; set short-lived access tokens.

# 12. Cookie limits & practical notes

* Browsers limit cookie size (~4KB per cookie) and number per domain (commonly ~20–50). Don’t store large objects.
* `SameSite` default behavior may vary; explicitly set `sameSite`.
* `sameSite: 'none'` requires `secure: true`. Modern browsers block `None` without `Secure`.

# 13. Example: auth middleware that checks HttpOnly cookie

```js
function requireAuth(req, res, next) {
  const token = req.cookies.access_token; // cookie-parser
  if (!token) return res.status(401).json({ message: 'Not authenticated' });

  try {
    const payload = verifyJwt(token); // your JWT verify function
    req.user = payload;
    next();
  } catch (err) {
    return res.status(401).json({ message: 'Invalid token' });
  }
}
```

# 14. Example full small app (ES module) — login sets cookie, protected route

```js
import express from 'express';
import cookieParser from 'cookie-parser';
import cors from 'cors';

const app = express();
app.use(express.json());
app.use(cookieParser('your-secret')); // if you plan to use signed cookies

app.use(cors({
  origin: 'http://localhost:5173', // frontend origin
  credentials: true
}));

// fake user check
app.post('/login', (req, res) => {
  const { username } = req.body;
  // create token (JWT or session id)
  const token = createJwt({ username }); // implement accordingly
  res.cookie('access_token', token, {
    httpOnly: true,
    secure: false,       // true in production (HTTPS)
    maxAge: 1000 * 60 * 60, // 1 hour
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

# 15. Common pitfalls & troubleshooting

* **Cookies not sent from browser**: check `credentials`, `sameSite`, `secure` flags, and that `origin` & CORS `credentials` are set correctly.
* **Cookie seems to be set but not present on next request**: verify domain/path, `secure` (if on http), and browser blocked it for `SameSite` policy.
* **Signed cookies show up in `req.cookies` not `req.signedCookies`**: you must set `{ signed: true }` when calling `res.cookie()` and initialized `cookie-parser` with the same secret.
* **Cookie not cleared**: ensure `path`/`domain` match when calling `res.clearCookie`.
* **Cookies overwritten unexpectedly**: check `path` and `domain`; cookies with the same name but different paths can coexist.

# 16. When to use cookies vs localStorage

* Use cookies (HttpOnly) for sensitive tokens because they are inaccessible to JS (protect against XSS).
* `localStorage`/`sessionStorage` is easier for client-accessible data but vulnerable to XSS; not recommended for auth tokens.

# 17. GDPR / consent note

If you are in GDPR jurisdiction or similar, user consent may be required before setting non-essential cookies (analytics, marketing). Implement a consent banner and do not set those cookies until consent is given.

# 18. Quick checklist before deploying

* Use HTTPS and `secure: true`.
* Use `httpOnly` for auth cookies.
* Set appropriate `sameSite`.
* Use short expirations for sensitive tokens.
* Store minimal data in cookies.
* Test cross-origin flows (frontend → API) with `credentials: 'include'`.
* Add CSRF protection.

---
