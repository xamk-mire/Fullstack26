# CSS Fundamentals: Style the Study Planner Page

In the HTML topic you built the **structure** of the page. Now you’ll use CSS to control **layout, spacing, typography, and basic responsiveness**.

You’ll style the **same `index.html`** by adding a CSS file (recommended) or a `<style>` block. The build-along keeps the CSS simple and readable—no frameworks yet—so you learn what Tailwind will later automate.

---

## What you’ll learn while styling

* How CSS rules target elements (selectors)
* The box model (margin, padding, border)
* Typography (font sizes, line height)
* Layout with Flexbox
* A simple responsive layout with a media query
* Basic form styling
* Focus styles (keyboard usability)

---

## Step 0: Add a CSS file

Create a file named `styles.css` in the same folder as `index.html`.

Then link it in your `<head>`:

```html
<link rel="stylesheet" href="styles.css">
```

Place it inside `<head>` (under `<title>` is fine).

---

## Step 1: Add a clean baseline (reset + base styles)

Put this at the top of `styles.css`:

```css
/* 1) Make sizing predictable */
*,
*::before,
*::after {
  box-sizing: border-box;
}

/* 2) Remove default browser spacing */
body {
  margin: 0;
}

/* 3) Base typography */
body {
  font-family: system-ui, -apple-system, Segoe UI, Roboto, Arial, sans-serif;
  line-height: 1.5;
}

/* 4) Make images behave nicely by default */
img {
  max-width: 100%;
  display: block;
}
```

### What this does

* `box-sizing: border-box` makes widths easier to reason about.
* Removing body margin prevents “mystery whitespace”.
* `system-ui` uses the user’s OS font for a clean default.

Refresh the page. You should see spacing change slightly.

---

## Step 2: Add a centered “container” layout

Add this:

```css
/* Page layout container */
main,
header,
footer {
  max-width: 900px;
  margin: 0 auto;
  padding: 16px;
}
```

### What this does

* `max-width` prevents overly wide lines on big screens
* `margin: 0 auto` centers the content
* `padding` keeps content away from the edges

---

## Step 3: Style the header + navigation

Add:

```css
header {
  border-bottom: 1px solid #ddd;
}

header h1 {
  margin: 0 0 8px 0;
  font-size: 1.8rem;
}

nav {
  display: flex;
  gap: 12px;
  flex-wrap: wrap;
}

nav a {
  text-decoration: none;
  color: #0b5bd3;
  padding: 6px 10px;
  border-radius: 8px;
}

nav a:hover {
  background: #eef4ff;
}

nav a:focus-visible {
  outline: 3px solid #0b5bd3;
  outline-offset: 2px;
}
```

### What to notice

* Flexbox makes the nav links align nicely and wrap if needed.
* `:focus-visible` makes keyboard navigation clearly visible.

---

## Step 4: Style sections and headings

Add:

```css
main {
  display: grid;
  gap: 16px;
}

section {
  border: 1px solid #e5e5e5;
  border-radius: 12px;
  padding: 16px;
  background: #fafafa;
}

h2 {
  margin: 0 0 12px 0;
  font-size: 1.3rem;
}

h3 {
  margin: 16px 0 8px 0;
  font-size: 1.05rem;
}
```

### Why this is useful

* Each section becomes a “card”.
* `main` uses a simple grid gap for spacing.

---

## Step 5: Make the task list look nicer

Add:

```css
ul {
  margin: 0;
  padding-left: 20px;
}

li {
  margin: 6px 0;
}
```

This keeps the list readable and not overly spaced.

---

## Step 6: Style the form (inputs + buttons)

Add:

```css
form {
  display: grid;
  gap: 12px;
  margin-top: 8px;
}

label {
  font-weight: 600;
}

/* Inputs */
input[type="text"],
input[type="date"] {
  width: 100%;
  padding: 10px 12px;
  border: 1px solid #cfcfcf;
  border-radius: 10px;
  font-size: 1rem;
}

input[type="text"]:focus,
input[type="date"]:focus {
  outline: 3px solid #b3d4ff;
  border-color: #0b5bd3;
}

/* Checkbox row looks better inline */
input[type="checkbox"] {
  transform: translateY(1px);
}

/* Buttons */
button {
  padding: 10px 14px;
  border: 1px solid #cfcfcf;
  border-radius: 10px;
  background: white;
  font-size: 1rem;
  cursor: pointer;
}

button:hover {
  background: #f2f2f2;
}

button:focus-visible {
  outline: 3px solid #0b5bd3;
  outline-offset: 2px;
}
```

At this point the form should look like a modern, readable UI.

---

## Step 7: Put the buttons on the same row (small HTML tweak)

Right now, your form has a `<div>` containing both buttons. Let’s give it a class so CSS can target it cleanly.

In `index.html`, find the button container and change:

```html
<div>
  <button type="submit">Add</button>
  <button type="reset">Clear</button>
</div>
```

to:

```html
<div class="button-row">
  <button type="submit">Add</button>
  <button type="reset">Clear</button>
</div>
```

Now add this CSS:

```css
.button-row {
  display: flex;
  gap: 10px;
  flex-wrap: wrap;
}

.button-row button[type="submit"] {
  border-color: #0b5bd3;
  background: #0b5bd3;
  color: white;
}

.button-row button[type="submit"]:hover {
  background: #094aa9;
}
```

Now you have a “primary” action button.

---

## Step 8: Make it responsive (mobile-friendly)

On wide screens, it’s nice to show “Today” and “Add task” side by side. On small screens, they should stack.

Replace your `main` layout rule:

```css
main {
  display: grid;
  gap: 16px;
}
```

with this:

```css
main {
  display: grid;
  gap: 16px;
}

/* On wider screens, use 2 columns */
@media (min-width: 800px) {
  main {
    grid-template-columns: 1fr 1fr;
    align-items: start;
  }
}
```

Try resizing the browser window:

* Below 800px: stacked sections
* Above 800px: two columns

---

## Step 9: Style the footer

Add:

```css
footer {
  border-top: 1px solid #ddd;
  color: #555;
}

footer a {
  color: inherit;
}
```

---

# Final `styles.css` (Complete)

If you want one full copy to compare against, here it is:

```css
*,
*::before,
*::after {
  box-sizing: border-box;
}

body {
  margin: 0;
  font-family: system-ui, -apple-system, Segoe UI, Roboto, Arial, sans-serif;
  line-height: 1.5;
}

img {
  max-width: 100%;
  display: block;
}

main,
header,
footer {
  max-width: 900px;
  margin: 0 auto;
  padding: 16px;
}

header {
  border-bottom: 1px solid #ddd;
}

header h1 {
  margin: 0 0 8px 0;
  font-size: 1.8rem;
}

nav {
  display: flex;
  gap: 12px;
  flex-wrap: wrap;
}

nav a {
  text-decoration: none;
  color: #0b5bd3;
  padding: 6px 10px;
  border-radius: 8px;
}

nav a:hover {
  background: #eef4ff;
}

nav a:focus-visible {
  outline: 3px solid #0b5bd3;
  outline-offset: 2px;
}

main {
  display: grid;
  gap: 16px;
}

@media (min-width: 800px) {
  main {
    grid-template-columns: 1fr 1fr;
    align-items: start;
  }
}

section {
  border: 1px solid #e5e5e5;
  border-radius: 12px;
  padding: 16px;
  background: #fafafa;
}

h2 {
  margin: 0 0 12px 0;
  font-size: 1.3rem;
}

h3 {
  margin: 16px 0 8px 0;
  font-size: 1.05rem;
}

ul {
  margin: 0;
  padding-left: 20px;
}

li {
  margin: 6px 0;
}

form {
  display: grid;
  gap: 12px;
  margin-top: 8px;
}

label {
  font-weight: 600;
}

input[type="text"],
input[type="date"] {
  width: 100%;
  padding: 10px 12px;
  border: 1px solid #cfcfcf;
  border-radius: 10px;
  font-size: 1rem;
}

input[type="text"]:focus,
input[type="date"]:focus {
  outline: 3px solid #b3d4ff;
  border-color: #0b5bd3;
}

input[type="checkbox"] {
  transform: translateY(1px);
}

button {
  padding: 10px 14px;
  border: 1px solid #cfcfcf;
  border-radius: 10px;
  background: white;
  font-size: 1rem;
  cursor: pointer;
}

button:hover {
  background: #f2f2f2;
}

button:focus-visible {
  outline: 3px solid #0b5bd3;
  outline-offset: 2px;
}

.button-row {
  display: flex;
  gap: 10px;
  flex-wrap: wrap;
}

.button-row button[type="submit"] {
  border-color: #0b5bd3;
  background: #0b5bd3;
  color: white;
}

.button-row button[type="submit"]:hover {
  background: #094aa9;
}

footer {
  border-top: 1px solid #ddd;
  color: #555;
}

footer a {
  color: inherit;
}
```

---

## Practical student tasks (to reinforce learning)

1. **Box model practice**

   * Increase the padding of `section` from `16px` → `24px`
   * Add `margin-top: 8px` to `nav`

2. **Typography**

   * Make `p` text slightly larger: `font-size: 1.05rem`
   * Make `small` text smaller: `font-size: 0.9rem`

3. **Add a “badge”**

   * Add a span inside the “Today” heading in HTML:

     ```html
     <h2>Today <span class="badge">3</span></h2>
     ```
   * Style `.badge` so it looks like a small pill

4. **Responsive tweak**

   * Change the breakpoint from `800px` to `650px` and see how it feels

