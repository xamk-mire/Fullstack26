## Exercise 1: “Web Sandbox” (HTML + CSS + JavaScript)

## Important!

Implement your solution inside your classroom exercises repository
  - Create a new folder inside the repo and name it as **Exercise-1**
  - Implement rest of the exercise inside the **Exercise-1** folder

### Goal

Create a small website that works as your personal **sandbox** for trying out Week 1 concepts. This is not about making something perfect — it’s about experimenting with **HTML elements**, **CSS styling**, and **JavaScript interactions** in one place.

By the end, you should have a single mini-site where you can:

* Add/modify HTML elements easily
* Try layout and styling techniques
* Connect JavaScript to the DOM and events
* Practice debugging using browser DevTools

---

## What you will build

A single-page website called **Web Sandbox Lab** with 4 sections:

1. **Elements Gallery (HTML)**
2. **Styling Playground (CSS)**
3. **JavaScript Interactions (JS)**
4. **Mini Challenge Zone (mix of all)**

You will implement required features, and then optionally add your own experiments.

---

## Setup

Create a folder:

```
web-sandbox/
  index.html
  styles.css
  script.js
```

In `index.html`, link the CSS and JS:

```html
<link rel="stylesheet" href="styles.css">
...
<script src="script.js"></script>
```

---

# Part A — HTML: Elements Gallery

### Required tasks

Create a section that demonstrates at least **10 different HTML elements**, including:

* One heading hierarchy (`h1`, `h2`, `h3`)
* A paragraph with **strong** and **em**
* An unordered list (`ul > li`)
* An ordered list (`ol > li`)
* A link (`a`)
* An image (`img`) (use any local image or a placeholder)
* A table (`table`, `tr`, `th`, `td`)
* A form with at least **3 different input types**
* A button
* A semantic layout structure (`header`, `nav`, `main`, `section`, `footer`)

### Suggested content idea

Use the elements to describe your “sandbox”, e.g.:

* A list of things you want to try in this course
* A table of “HTML tag experiments”
* A form for “add a new experiment”

---

# Part B — CSS: Styling Playground

### Required tasks

Create a section where you try **at least 8 CSS features**.

Must include:

* Use of **class selectors** and **id selectors**
* Box model: `margin`, `padding`, `border`
* `border-radius`
* A hover effect (`:hover`)
* A focus effect (`:focus` or `:focus-visible`)
* Flexbox OR Grid (at least one)
* A responsive change using **one media query**
* At least one “theme” or “style switch” using a CSS class (used later in JS)

### Suggested ideas

* Style a “card” component
* Create a navigation bar
* Create a two-column layout that becomes one column on small screens
* Add a visible focus outline to inputs/buttons

---

# Part C — JavaScript: Interactions Zone

Create a section titled **JavaScript Interactions** and implement these features:

### Required features (core)

1. **Counter**

   * Buttons: “+”, “−”, “Reset”
   * Display the current value in the page

2. **Live text preview**

   * An input field
   * A preview area that updates as you type

3. **Add items to a list**

   * Input + “Add” button
   * Add items as `<li>` elements to a list

4. **Toggle visibility**

   * Button that shows/hides a “secret message” or a div

### Required features (DOM + events)

* At least one `click` event listener
* At least one `input` event listener
* At least one use of `classList.add/remove/toggle`
* At least one use of `createElement` and `append`

### Optional features (for exploration)

* Store something in `localStorage`
* Sort the list
* Delete items from the list
* Add basic validation messages

---

# Part D — Mini Challenge Zone (mix everything)

Choose **one** mini-feature to build that combines HTML + CSS + JS.

Pick one:

* **Mini To-Do**: Add tasks, mark done, remove tasks
* **Color picker**: Choose a color and apply it to a preview box
* **Tabs**: Clicking buttons shows different content sections
* **Accordion FAQ**: Expand/collapse questions

This section is where you’re encouraged to “try something new”.

---

## Submission checklist (what to turn in)

  - ✅ A working `index.html`, `styles.css`, `script.js`
  - ✅ All required HTML elements demonstrated
  - ✅ All required CSS features used
  - ✅ All required JS interactions working
