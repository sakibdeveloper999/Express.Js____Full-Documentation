
---

## 🧠 Step 1: What is Templating in Express.js?

**Templating** means creating **HTML pages dynamically** on the server using a **template engine**.

When a request comes from a browser, Express can:

* take a **template file** (like `.ejs`, `.pug`, `.hbs`)
* fill it with **data from the server**
* and send the **final HTML** page to the client.

So instead of writing 100s of HTML files, you write **one template** that changes based on data.

---

### 💡 Simple Example (Concept)

You have a template like:

```html
<h1>Hello <%= name %></h1>
```

If the user’s name is “Sakib”, Express fills it:

```html
<h1>Hello Sakib</h1>
```

That’s templating — dynamic HTML generation.

---

## ⚙️ Step 2: Why use Templating?

Templating is used for:

1. **Dynamic websites** – generate pages with data (users, blogs, products).
2. **Reusability** – use one layout for all pages (header/footer once).
3. **Server-side rendering (SSR)** – better SEO and faster first load.
4. **Clean separation** – logic (Node.js code) is separate from presentation (HTML).

---

## 🧩 Step 3: How Express.js Uses Template Engines

Express does not render HTML by itself — it uses a **template engine**.

The workflow is:

1. Install a template engine (EJS, Pug, or Handlebars).
2. Configure it using `app.set()`.
3. Create `.ejs` (or other) files in the `views` folder.
4. Use `res.render()` to render that template.

---

## ⚒️ Step 4: Basic Setup Example (EJS)

### 1️⃣ Install Dependencies

```bash
npm install express ejs
```

### 2️⃣ Project Structure

```
project/
 ├── app.js
 └── views/
      └── index.ejs
```

### 3️⃣ app.js

```js
const express = require('express');
const app = express();

// Step 1: Set EJS as the view engine
app.set('view engine', 'ejs');

// Step 2: Define a route
app.get('/', (req, res) => {
  // Step 3: Render a template and send data
  res.render('index', { title: 'Home Page', name: 'Sakib' });
});

// Step 4: Start the server
app.listen(3000, () => console.log('Server running on http://localhost:3000'));
```

### 4️⃣ views/index.ejs

```ejs
<!DOCTYPE html>
<html>
<head>
  <title><%= title %></title>
</head>
<body>
  <h1>Welcome, <%= name %>!</h1>
</body>
</html>
```

👉 When you visit `http://localhost:3000`,
you’ll see: **“Welcome, Sakib!”**

That’s templating in action.

---

## 🧠 Step 5: How `res.render()` Works

When you call:

```js
res.render('index', { title: 'Home', name: 'Sakib' });
```

Here’s what Express does internally:

1. Looks inside the `views/` folder for `index.ejs`.
2. Sends `{ title, name }` to that file.
3. The EJS engine replaces `<%= %>` with real data.
4. Sends the generated HTML as the HTTP response.

So it transforms:

```html
<h1>Welcome, <%= name %>!</h1>
```

→

```html
<h1>Welcome, Sakib!</h1>
```

---

## 🧩 Step 6: Using Loops, Conditions & Includes

Templating lets you add logic directly in HTML.

### ✅ Example 1 — Loop

```ejs
<ul>
  <% users.forEach(user => { %>
    <li><%= user %></li>
  <% }) %>
</ul>
```

### ✅ Example 2 — Conditional

```ejs
<% if (loggedIn) { %>
  <p>Hello, <%= name %>!</p>
<% } else { %>
  <p>Please log in.</p>
<% } %>
```

### ✅ Example 3 — Include Partial

`views/header.ejs`

```ejs
<header><h1>My Site</h1></header>
```

`views/index.ejs`

```ejs
<%- include('header') %>
<p>Welcome, <%= name %></p>
```

✅ **Result:** The header file’s content is inserted wherever you include it.

---

## 🧱 Step 7: Using Layouts (Reusable Structure)

You can reuse one **layout** for all pages.

### layout.ejs

```ejs
<html>
  <head><title><%= title %></title></head>
  <body>
    <%- body %>
  </body>
</html>
```

### index.ejs

```ejs
<h1>Welcome <%= name %></h1>
```

Then you can render with `express-ejs-layouts` package or manually include parts.

This keeps your code **DRY (Don’t Repeat Yourself)**.

---

## 💼 Step 8: Real-Life Use Cases

| Use Case               | Description                                            |
| ---------------------- | ------------------------------------------------------ |
| 📰 **Blog Website**    | Render posts dynamically from a database.              |
| 🛒 **E-commerce Site** | Show product list using a loop.                        |
| 👨‍💻 **Admin Panel**  | Render tables of users/orders dynamically.             |
| 📩 **Email Templates** | Use templates to generate HTML emails.                 |
| 📑 **Reports**         | Create server-side reports or PDFs with template HTML. |

---

## ⚙️ Step 9: Other Template Engines (Optional Choices)

| Engine               | Syntax Style      | Highlights                       |
| -------------------- | ----------------- | -------------------------------- |
| **EJS**              | HTML + `<% %>` JS | Simple, closest to raw HTML      |
| **Pug**              | Indentation-based | Minimal syntax, clean structure  |
| **Handlebars (HBS)** | `{{ }}` style     | Safe, supports layouts & helpers |

All engines do the same thing — render data into HTML — just with different syntax.

---

## 🔐 Step 10: Security Tips

1. **Never trust raw user input** — always escape it.
   Use `<%= %>` (escaped) not `<%- %>` unless you know the content is safe.
2. **Avoid XSS (Cross-Site Scripting)** — do not inject unsanitized HTML.
3. **Use partials/layouts** — reduces repetition and accidental mistakes.
4. **Enable caching in production**:

   ```js
   app.set('view cache', true);
   ```

   This improves speed.

---

## 🚀 Step 11: Custom Template Engine (Advanced)

You can even make your own template engine:

```js
const fs = require('fs');
const express = require('express');
const app = express();

app.engine('txt', (filePath, options, callback) => {
  fs.readFile(filePath, (err, content) => {
    if (err) return callback(err);
    const rendered = content.toString().replace('#title#', options.title);
    return callback(null, rendered);
  });
});

app.set('views', './views');
app.set('view engine', 'txt');

app.get('/', (req, res) => {
  res.render('index', { title: 'Custom Template Engine!' });
});

app.listen(3000);
```

`views/index.txt`

```
<h1>#title#</h1>
```

✅ Output:

```
<h1>Custom Template Engine!</h1>
```

This shows how flexible Express is.

---

## ⚡ Step 12: Summary Table

| Step | Description                                    |
| ---- | ---------------------------------------------- |
| 1    | Install a template engine                      |
| 2    | Set it using `app.set('view engine', 'ejs')`   |
| 3    | Create `.ejs` files inside `/views`            |
| 4    | Pass data using `res.render('view', { data })` |
| 5    | Use loops, conditions, and includes            |
| 6    | Reuse layouts and partials                     |
| 7    | Escape output for security                     |
| 8    | Cache templates in production                  |

---

## 🏁 Final Summary (in one sentence)

**Templating in Express.js** means using a template engine (like EJS, Pug, or Handlebars) to dynamically combine data and HTML on the server before sending it to the client — allowing reusable, data-driven, and SEO-friendly pages with clean code separation.

---
