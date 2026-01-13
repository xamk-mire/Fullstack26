# HTML Fundamentals 

HTML (HyperText Markup Language) is the language that gives web pages their **structure and meaning**. If a webpage were a house:

* **HTML** is the floor plan and the rooms (structure: “this is a heading”, “this is navigation”, “this is a form”)
* **CSS** is the paint, furniture, and decoration (appearance)
* **JavaScript** is the electricity and moving parts (behavior)

In this topic, you’ll learn how to write HTML that is:

* **Correct** (valid page structure)
* **Readable** (clear organization)
* **Semantic** (uses the right elements for the job)
* **Accessible** (works for keyboard users and screen readers)

Even later, when you build with React, the browser still ultimately receives HTML. If you understand HTML well, React will feel far less mysterious.

---

## What you’re building in this topic (build-along)

Throughout this introduction, you’ll build a small “mini app” page called:

**Study Planner (HTML-only)**

It will include:

* A page header and navigation
* A “Today’s tasks” list
* A small form to add a new task (not functional yet—functionality comes in JavaScript topic)
* A footer

You can build it in one file: `index.html`.

### How to run it

1. Create a folder (e.g., `html-fundamentals`)
2. Create a file named `index.html`
3. Open the file in a browser (double click), or use “Open with Live Server” if you have that extension

---

## 1) HTML is made of elements

An HTML **element** usually has:

* an opening tag: `<p>`
* content: `Hello`
* a closing tag: `</p>`

```html
<p>Hello</p>
```

Some elements don’t wrap content (void elements), such as:

```html
<img src="logo.png" alt="Logo">
<input type="text">
```

### Build-along step

Open your `index.html` and add:

```html
<p>Study Planner is loading…</p>
```

Open it in the browser. You should see that sentence on the page.

---

## 2) A proper HTML document structure

Real pages follow this structure:

```html
<!doctype html>
<html lang="en">
  <head>
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <title>Study Planner</title>
  </head>
  <body>
    <!-- Visible content goes here -->
  </body>
</html>
```

### What these parts mean

* `<!doctype html>`: tells the browser to use modern HTML rules
* `<html lang="en">`: language of the document (important for accessibility)
* `<head>`: metadata (title, encoding, CSS links…)
* `<body>`: the content you see on the page

### Build-along step

Replace your file with this complete skeleton:

```html
<!doctype html>
<html lang="en">
  <head>
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <title>Study Planner</title>
  </head>
  <body>
    <p>Study Planner is loading…</p>
  </body>
</html>
```

Refresh the page. Nothing will look different yet—but now your document is “real”.

---

## 3) Semantic HTML: choosing elements that describe meaning

You *can* build everything with `<div>`, but you *shouldn’t*. Semantic tags tell the browser (and assistive technologies) what something **is**.

Common semantic elements:

* `<header>`: top of the page/section
* `<nav>`: navigation links
* `<main>`: main page content
* `<section>`: a meaningful group of content
* `<footer>`: bottom section

### Why this matters

Semantic HTML:

* improves accessibility (screen readers understand page regions)
* improves maintainability (humans understand your code faster)
* helps later when styling/layout becomes complex

### Build-along step: add page layout regions

Replace the `<p>` in `<body>` with:

```html
<header>
  <h1>Study Planner</h1>
  <nav>
    <a href="#today">Today</a>
    <a href="#add">Add task</a>
  </nav>
</header>

<main>
  <section id="today">
    <h2>Today</h2>
    <p>Your tasks for today will appear here.</p>
  </section>

  <section id="add">
    <h2>Add a task</h2>
    <p>A form will go here soon.</p>
  </section>
</main>

<footer>
  <small>Built in Sprint 1</small>
</footer>
```

Refresh. You should now see:

* a big title (h1)
* navigation links
* two sections
* a footer

Click the navigation links—notice how the page jumps to the matching section. That’s because `href="#today"` links to `id="today"`.

---

## 4) Headings create structure (not just big text)

Headings are not for decoration. They define the “outline” of a page.

* `<h1>`: page title (usually one per page)
* `<h2>`: section titles
* `<h3>`: subsections inside sections

### Build-along improvement

Inside your “Today” section, add a subsection heading:

```html
<h3>Priority</h3>
<p>Focus on the most important tasks first.</p>
```

This gives your content a clear hierarchy.

---

## 5) Lists: the correct way to represent repeated items

Lists are extremely common in web apps (tasks, messages, products, comments).

### Unordered list

```html
<ul>
  <li>Read lecture notes</li>
  <li>Complete HTML exercises</li>
  <li>Commit changes to Git</li>
</ul>
```

### Build-along step: create a tasks list

In the “Today” section, replace the paragraph with a list:

```html
<ul>
  <li>Read lecture notes</li>
  <li>Complete HTML exercises</li>
  <li>Commit changes to Git</li>
</ul>
```

Refresh and check: each task should appear as a bullet point.

---

## 6) Links and paths (how files connect)

Links (`<a>`) are one of the defining features of the web.

```html
<a href="https://example.com">External link</a>
```

For internal navigation, we used `#id`.

Later, when we build multi-page apps and routing, the idea is similar: the browser navigates to a new “location”.

### Build-along mini task

Add a link in your footer to your course page (or any URL):

```html
<footer>
  <small>
    Built in Sprint 1 · <a href="https://example.com">Course page</a>
  </small>
</footer>
```

---

## 7) Forms: collecting user input

Forms allow users to submit data. Even before JavaScript, forms help you understand the core idea: **inputs + submit**.

### Key elements

* `<form>`: groups inputs
* `<label>`: describes an input (important!)
* `<input>`: a field
* `<button>`: trigger actions (often submit)

### Build-along step: add the “Add task” form

Replace the paragraph in the “Add a task” section with:

```html
<form>
  <div>
    <label for="title">Task title</label><br>
    <input id="title" name="title" type="text" placeholder="e.g., Finish exercises" required>
  </div>

  <div>
    <label for="due">Due date</label><br>
    <input id="due" name="due" type="date">
  </div>

  <div>
    <button type="submit">Add</button>
    <button type="reset">Clear</button>
  </div>
</form>
```

Try it:

* Click the input fields and type
* Try submitting with an empty title (the browser should block it because of `required`)
* Click “Clear” and watch the fields reset

### Notes

* `for="title"` matches `id="title"`: this links the label to the input.
* `name="title"` is the key used when form data is sent.
* `required` uses the browser’s built-in validation.

(We used `<br>` here just to keep it simple visually. In the CSS topic, you’ll learn proper layout without `<br>`.)

---

## 8) Buttons: avoid a common surprise

Inside a form:

* `<button>` defaults to `type="submit"` (if you don’t specify)
* This can cause accidental submissions later

We explicitly set:

* `type="submit"` for Add
* `type="reset"` for Clear

Good habit early.

---

## 9) Attributes: small details that matter

Attributes provide extra information:

* `id`: unique identifier (used by labels, navigation, and JS later)
* `class`: used by CSS (and later Tailwind)
* `placeholder`: hint text
* `required`: validation
* `type`: changes input behavior (date/email/password…)

### Build-along mini task

Add a checkbox to your form:

```html
<div>
  <label>
    <input type="checkbox" name="important">
    Mark as important
  </label>
</div>
```

This demonstrates a slightly different pattern: wrapping the input inside the label.

---

## 10) Your final build-along result (complete file)

If you want to compare, your `index.html` could look like this:

```html
<!doctype html>
<html lang="en">
  <head>
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <title>Study Planner</title>
  </head>
  <body>
    <header>
      <h1>Study Planner</h1>
      <nav>
        <a href="#today">Today</a>
        <a href="#add">Add task</a>
      </nav>
    </header>

    <main>
      <section id="today">
        <h2>Today</h2>

        <h3>Tasks</h3>
        <ul>
          <li>Read lecture notes</li>
          <li>Complete HTML exercises</li>
          <li>Commit changes to Git</li>
        </ul>

        <h3>Priority</h3>
        <p>Focus on the most important tasks first.</p>
      </section>

      <section id="add">
        <h2>Add a task</h2>

        <form>
          <div>
            <label for="title">Task title</label><br>
            <input id="title" name="title" type="text" placeholder="e.g., Finish exercises" required>
          </div>

          <div>
            <label for="due">Due date</label><br>
            <input id="due" name="due" type="date">
          </div>

          <div>
            <label>
              <input type="checkbox" name="important">
              Mark as important
            </label>
          </div>

          <div>
            <button type="submit">Add</button>
            <button type="reset">Clear</button>
          </div>
        </form>
      </section>
    </main>

    <footer>
      <small>
        Built in Sprint 1 · <a href="https://example.com">Course page</a>
      </small>
    </footer>
  </body>
</html>
```

---

## Small practical exercise (student task)

Try these improvements (no CSS needed yet):

1. **Add one more navigation link** to a new section called “This Week”.

   * Add a `<section id="week">...</section>`
   * Add `<a href="#week">This Week</a>` to `<nav>`

2. In “This Week”, add an **ordered list** of steps you’ll take to succeed in the course.

3. Add an input to the form:

   * a `<select>` dropdown for “Course module” with 3 options (e.g., HTML, CSS, JavaScript)

4. Add a short paragraph under the form:

   * Explain (in your own words) what `required`, `id`, and `name` do.

---

## What you should understand after this topic

By the end, you should be able to:

* Write a valid HTML document structure
* Use semantic elements to organize content
* Use headings and lists correctly
* Create accessible forms with labels and inputs
* Explain how attributes like `id`, `name`, and `required` work
