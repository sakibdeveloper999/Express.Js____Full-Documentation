

# EJS — Full documentation & practical guide (step-by-step)

---

## What is EJS?

EJS (Embedded JavaScript) is a minimal, fast templating language that lets you embed JavaScript code inside HTML to produce server-rendered HTML. It’s intentionally simple: templates are plain HTML files with small JavaScript tags. EJS is often used with Express to render dynamic pages on the server.

Key features:

* Small syntax surface (easy to learn if you know JS).
* Automatic HTML-escaping by default.
* Works as both a library (render strings/files) and as view engine for Express.
* Fast and lightweight — good for server-rendered HTML, emails, admin UIs.

---

## Basic concepts

* **Views/templates**: `.ejs` files containing HTML and embedded JS tags.
* **Data/locals**: Objects passed to templates to render dynamic values.
* **Rendering**: Converting template + data into HTML string (server side).
* **Partials/includes**: Reusable template fragments (header/footer).
* **Escaping**: EJS escapes output by default to protect against XSS.

---

## EJS syntax (the essential tags)

* Output escaped (safe):

```ejs
<%= variable %>
```

* Output unescaped (raw HTML — dangerous with untrusted data):

```ejs
<%- htmlString %>
```

* Run JS code (no output directly):

```ejs
<% if (user) { %>
  Hello
<% } %>
```

* Single-line comment in template (not output):

```ejs
<% /* comment */ %>
```

* Raw JS control flow (loops, conditions):

```ejs
<% for (let i = 0; i < items.length; i++) { %>
  <li><%= items[i] %></li>
<% } %>
```

Notes:

* Use `<%= %>` for any user input or untrusted content — it escapes HTML entities (`<`, `>`, `&`, `"`, `'`, etc).
* Use `<%- %>` only for content you fully trust (e.g., sanitized HTML or static markup).

---

## Installing & basic Express integration

1. Create project and install:

```bash
npm init -y
npm install express ejs
```

2. Configure Express to use EJS:

```js
const express = require('express');
const app = express();

// Set EJS as the view engine
app.set('view engine', 'ejs');

// By default, Express looks in ./views; you can change it:
// app.set('views', path.join(__dirname, 'templates'));

app.get('/', (req, res) => {
  res.render('index', { title: 'Home', user: { name: 'Sakib' }});
});

app.listen(3000);
```

3. Example `views/index.ejs`:

```ejs
<!doctype html>
<html>
<head>
  <title><%= title %></title>
</head>
<body>
  <h1>Welcome, <%= user.name %>!</h1>
</body>
</html>
```

`res.render(viewName, locals)` finds `views/viewName.ejs`, renders and sends HTML.

---

## Core API (EJS as a library)

EJS can be used without Express. The core functions you’ll use:

* `ejs.render(templateString, data, options)`
  Renders a template string synchronously to HTML.

* `ejs.renderFile(filename, data, options, callback)`
  Renders a file (async with callback or returning a promise depending on usage).

* `ejs.compile(templateString, options)`
  Compiles a template to a reusable function: `const fn = ejs.compile(tpl); const html = fn(data)`.

* Some implementations/support also expose `renderFile` returning a Promise when callback is omitted.

**Basic examples:**

```js
const ejs = require('ejs');

const tpl = 'Hello <%= name %>!';
const html = ejs.render(tpl, { name: 'Sakib' });
// html === 'Hello Sakib!'

// compile
const fn = ejs.compile('Sum: <%= a + b %>');
console.log(fn({ a: 1, b: 2 })); // 'Sum: 3'
```

---

## Important options (common & practical)

EJS exposes options you can pass to `render`, `renderFile`, or `compile`. Key options you’ll use in real projects:

* `filename` — hint for error stack traces and includes. Set this when rendering raw strings if you want meaningful stack traces and include resolution.
* `cache` — enable template caching (true/false). In production, caching improves performance.
* `delimiter` — change the special delimiter character (default is `<% %>` style using `%` delimiter). Useful when another language conflicts with `<%`.
* `rmWhitespace` — remove/trims safe whitespace between tags (helps minify output).
* `compileDebug` — when `false`, reduces compiled output size (turn off in production).
* `localsName` — name of the locals object when compiled for client use (rare).
* `context` — provide the context object for compiled templates when using `with` is disabled.
* `async` — some EJS versions support async templates (await inside templates). Check your ejs version for `async` support.

> Implementation note: the exact option names and availability may vary by ejs version — use the options most relevant to your version. `filename`, `cache`, `delimiter` and `rmWhitespace` are widely supported.

---

## Partials & `include`

**include** is how you reuse fragments.

Simple use:

```ejs
<%- include('partials/header') %>

<h1>Page body</h1>

<%- include('partials/footer') %>
```

Notes:

* `include()` inserts the included file at render time.
* Using `<%- include(...) %>` (unescaped include) is typical because `include` returns HTML.
* Include resolution depends on `filename`/`views`/`root` options; if you get "file not found" check `views` directory and pass `filename` when rendering strings.
* Local variables from parent are accessible inside includes.

---

## Layouts (patterns)

EJS has no built-in block layout system like some templating engines. Common patterns:

### 1) Manual layout injection

Render the body first into a string and then inject into layout:

```js
const body = ejs.renderFile('views/posts.ejs', { posts });
const html = ejs.renderFile('views/layout.ejs', { body });
```

This is manual and rarely convenient.

### 2) Use a small layout helper package (common in many projects)

There are small middleware/utility packages that add layout behavior (e.g., set a `layout` and use `<%- body %>`). If you prefer zero dependencies, implement middleware that sets `res.locals.layout` and call a helper.

### 3) Partial-based approach

Put header/footer in partials and include them in every page:

```ejs
<%- include('partials/header') %>
<main> ... </main>
<%- include('partials/footer') %>
```

This is straightforward and widely used.

---

## Helpers: `app.locals` and `res.locals`

In Express:

* `app.locals` — global helpers available in all templates (site name, utility functions).

```js
app.locals.siteName = 'My Site';
app.locals.formatDate = d => d.toLocaleDateString();
```

* `res.locals` — request-scoped locals (current user, flash messages).

```js
app.use((req, res, next) => {
  res.locals.currentUser = req.user;
  next();
});
```

Call templates normally: `<%= formatDate(post.date) %>`

---

## Escaping & security (XSS protection)

* **Default**: `<%= %>` escapes HTML characters. This is your first-line defence against XSS.
* **Do not** use `<%- %>` with strings containing user-provided HTML unless you sanitized them and are confident they’re safe.
* **Sanitization**: If you must accept HTML input, sanitize it server-side with an HTML sanitizer (strip scripts, dangerous attributes). EJS itself doesn’t sanitize input beyond escaping.
* **CSP (Content Security Policy)**: use CSP headers to limit script execution sources.
* **HTTP headers**: set `X-Frame-Options`, `X-Content-Type-Options`, etc.
* **Server-side validation**: always validate and escape data before saving and before rendering.

---

## Performance & caching

* In production, **enable template caching**. Express typically caches compiled templates in production mode.
* Avoid rendering dozens of separate templates per request — join or pre-render partials if needed.
* For very heavy pages, consider:

  * caching rendered page fragments,
  * using `ejs.compile` to compile once then reuse function,
  * offloading heavy computation to workers prior to render.

Example: compile once at startup:

```js
const compiled = ejs.compile(fs.readFileSync('views/report.ejs', 'utf8'), { filename: 'views/report.ejs' });
app.get('/report', (req, res) => {
  res.send(compiled({ data: someData }));
});
```

---

## Debugging templates

* Use `filename` option so stack traces refer to real file names:

```js
ejs.render(str, data, { filename: path.join(__dirname, 'views', 'file.ejs') });
```

* Turn `compileDebug: true` (development) to get better stack traces.
* Put `console.log()` inside server code to inspect locals before render.
* Build small templates and add content step by step to isolate errors.
* Syntax errors usually point to line numbers in compiled function — `filename` makes them meaningful.

---

## Advanced topics

### 1) Server-side rendering & client templates

EJS can compile templates to functions that can be shipped to client-side JS (if you want client rendering). You’d compile with `client: true` (if supported) and send the client template and runtime. Use cautiously.

### 2) Async helpers & includes

Some EJS versions support `async` templates or async includes so you can `await` inside template. Check your version; enabling `async` may be necessary and changes the compiled function signature. Use async templates sparingly — heavy async logic ideally belongs in the controller.

### 3) Custom delimiters

If `<%` conflicts (e.g., another template engine or framework), change `delimiter`:

```js
ejs.render(str, data, { delimiter: '?' });
// new tags: <?= ... ?> etc (example — check exact delimiter usage)
```

Be careful: changing delimiter changes all template tags.

### 4) File includes resolution & `root`/`views`

When using includes, EJS resolves paths relative to the `filename` of the current template or to `root` / `views` options. If you render template strings programmatically, set `filename` option to help include resolution.

---

## Common pitfalls & gotchas

* Using `<%- %>` with untrusted user input → XSS.
* Failing to pass expected locals → `undefined` errors in templates.
* Not setting `views` path correctly → "template not found".
* Trying to do heavy business logic inside templates — templates should be for presentation.
* Forgetting to set `filename` when rendering strings → hard-to-read stack traces and include resolution problems.

---

## Testing templates

* Treat templates like functions: test them by compiling and calling with representative data, including edge cases (empty arrays, missing fields).
* Example using a test runner:

```js
const assert = require('assert');
const ejs = require('ejs');
const tpl = '<h1><%= title %></h1>';

const html = ejs.render(tpl, { title: 'Test' });
assert(html.includes('<h1>Test</h1>'));
```

* Snapshot tests: render template and compare to expected HTML string or use a DOM parser to assert presence of specific nodes/attributes.

---

## Migration tips (if coming from other engines)

* Replace template-specific syntax with EJS tags:

  * `{{var}}` → `<%= var %>`
  * Logic blocks like `{% if ... %}` → `<% if (...) { %>`
* Break pages into partials and include them using `<%- include(...) %>`.
* Use `app.locals` for helpers previously provided by other engines’ helper systems.

---

## Example: Full small project (copy-paste runnable)

File structure:

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

`package.json` (minimal):

```json
{
  "name": "ejs-doc-demo",
  "version": "1.0.0",
  "main": "app.js",
  "dependencies": {
    "express": "^4.0.0",
    "ejs": "^3.0.0"
  },
  "scripts": {
    "start": "node app.js"
  }
}
```

`app.js`

```js
const express = require('express');
const path = require('path');
const app = express();
const port = process.env.PORT || 3000;

// View engine
app.set('view engine', 'ejs');
app.set('views', path.join(__dirname, 'views'));

// Static files
app.use(express.static(path.join(__dirname, 'public')));

// Globals
app.locals.siteName = 'EJS Demo Store';
app.locals.formatCurrency = v => '$' + Number(v).toFixed(2);

// Simple middleware to set request-scoped locals
app.use((req, res, next) => {
  res.locals.currentPath = req.path;
  // example current user (in real app set from session/auth)
  res.locals.user = { name: 'Guest' };
  next();
});

// Routes
app.get('/', (req, res) => {
  res.render('index', { title: 'Welcome' });
});

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

`views/partials/header.ejs`

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

`views/partials/footer.ejs`

```ejs
  </main>
  <footer>
    <small>&copy; <%= new Date().getFullYear() %> <%= siteName %></small>
  </footer>
</body>
</html>
```

`views/index.ejs`

```ejs
<%- include('partials/header') %>

<section>
  <h2><%= title %></h2>
  <p>Hello, <%= user.name %> — this is a demo home page rendered with EJS.</p>
</section>

<%- include('partials/footer') %>
```

`views/products.ejs`

```ejs
<%- include('partials/header') %>

<section>
  <h2><%= title %></h2>
  <ul>
    <% if (products.length) { %>
      <% products.forEach(p => { %>
        <li>
          <strong><%= p.name %></strong> — <%= formatCurrency(p.price) %>
        </li>
      <% }) %>
    <% } else { %>
      <li>No products found.</li>
    <% } %>
  </ul>
</section>

<%- include('partials/footer') %>
```

`public/css/styles.css` (basic):

```css
body { font-family: system-ui, sans-serif; margin: 0; padding: 0; }
header { background:#222; color:#fff; padding: 1rem; }
header a { color: #fff; text-decoration: none; margin-right: 1rem; }
main { padding: 1rem; }
.active { text-decoration: underline; }
```

Run:

```bash
npm install
npm start
# open http://localhost:3000
```

---

## Example real-world patterns & recipes

1. **Pagination partial**

   * Build a `partials/pagination.ejs` and pass `{page, totalPages, baseUrl}`; include logic to disable previous/next links.

2. **Form handling with old values & errors**

   * On POST, validate server-side and render same form template with `errors` and `old` locals:

     ```ejs
     <input name="email" value="<%= (old && old.email) || '' %>" />
     <% if (errors && errors.email) { %>
       <small class="error"><%= errors.email %></small>
     <% } %>
     ```

3. **Accessible components**

   * Keep HTML semantic in templates and prefer server-generated ARIA attributes when needed.

4. **Email templates**

   * Use EJS to generate HTML emails; keep separate `views/emails` folder and compile with `ejs.renderFile` to get email HTML string.

---

## FAQ (short)

* **Q: Can I use JS expressions in templates?**
  A: Yes. Use any JS expressions (e.g., `<%= items.length ? 'yes' : 'no' %>`). Keep templates readable—complex logic belongs in controllers/helpers.

* **Q: Does EJS escape by default?**
  A: Yes—`<%= %>` escapes. `<%- %>` outputs raw HTML.

* **Q: Can I use ESM `import` with ejs?**
  A: You can if your Node environment is configured for ESM and the packages support it. Typical usage in CommonJS as shown is most common.

* **Q: Is EJS good for large apps?**
  A: Yes — with patterns: partials, helpers via `app.locals`, caching, and clean separation of logic & presentation.

* **Q: How to include dynamic partials?**
  A: Use includes with variable paths carefully. Example:

  ```ejs
  <%- include('partials/' + widgetName) %>
  ```

  Provide defensive checks server-side to avoid path injection.

---

## Best practices checklist (again — condensed)

* Always escape untrusted data (`<%= %>`).
* Keep templates thin — no heavy business logic.
* Use partials for repeated markup.
* Add helpers via `app.locals` or separate helper modules.
* Set `filename` when rendering strings for better errors and include resolution.
* Enable caching in production.
* Test templates for edge cases.
* Use server-side sanitization for any HTML from users.
* Use meaningful file and folder naming: `views/partials`, `views/emails`, `views/layouts`.

---
