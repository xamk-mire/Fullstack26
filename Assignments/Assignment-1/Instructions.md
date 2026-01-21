# **Assignment 1: Task Tracker (Vanilla JS) — (No Server)**
---

## Introduction 

Modern web applications are often built with tools like React, frameworks, bundlers, and local development servers. However, all of these tools are built on top of three fundamental technologies:

- **HTML** — describes _what is on the page_
    
- **CSS** — describes _how it looks_
    
- **JavaScript** — describes _how it behaves_
    

In this assignment, you will deliberately work **without frameworks, build tools, or a local server**. You will create a Task Tracker that runs simply by double-clicking `index.html`. This approach:

- Forces you to understand how the browser actually loads and executes files
    
- Makes you think carefully about DOM manipulation
    
- Makes the relationship between “data” (your tasks array) and “UI” (what the user sees) very explicit
    
- Exposes common problems that frameworks like React are designed to solve
    

In later lectures and assignments, you will revisit this project conceptually and compare your manual implementation with how React would handle the same problems more elegantly.

---

## Core learning goals

After completing this assignment, you should be able to:

### Web fundamentals

- Structure a page using **semantic HTML** (`header`, `main`, `nav`, `section`, `footer`)
    
- Create responsive layouts using **Flexbox or CSS Grid**
    
- Apply consistent spacing, visual hierarchy, and basic accessibility principles
    

### JavaScript and the browser

- Read and modify the DOM using `document.getElementById`, `querySelector`, etc.
    
- Generate UI dynamically using JavaScript
    
- Handle user input via:
    
    - `click`
        
    - `submit`
        
    - `change`
        
- Use **event delegation** instead of attaching many individual event listeners
    
- Persist data using `localStorage`
    

### Application thinking (important for React later)

- Distinguish between:
    
    - **State** (your tasks array)
        
    - **Rendering** (what is shown in the UI)
        
- Understand why keeping state and UI separate is important
    
- Experience the complexity that arises when an app grows beyond a single page

---

## Important technical constraint (non-server approach)

🚫 You **must NOT use:**

- `type="module"` (technically you could, but then you most likely are going to be needing a server (CORS issues))
    
- `import` / `export` 
    
- Node.js, Vite, Webpack, Parcel, or any local server
    
- External JavaScript libraries or frameworks (vanilla implementation this time around -> feel the pain approach)
    

✅ Your app **must work by simply double-clicking `index.html` in a file explorer.**

This means:

- All JavaScript must be loaded as regular scripts (not ES modules)
    
- Scripts must be loaded in the correct order in `index.html`
    
- Functions must be attached to `window` if they are shared across files


---

# ✅ **STEP 1 — Project Setup: Structure and Responsibility**

### 🎯 Goal

Create a clear project structure and define responsibilities for each file.

### Required folder structure (you may add files later, but start here)

```
task-tracker/
  index.html
  styles/
    main.css
  scripts/
    storage.js
    ui.js
    app.js
```

### What each file is responsible for

**`index.html` — The shell of your app**

- Contains only the basic page structure:
    
    - `header`
        
    - `nav`
        
    - `main` (with an element like `<main id="app"></main>`)
        
    - `footer`
        
- Loads your three JavaScript files **in this order**:
    
    1. `storage.js`
        
    2. `ui.js`
        
    3. `app.js`
        

```html
<script src="scripts/storage.js"></script>
<script src="scripts/ui.js"></script>
<script src="scripts/app.js"></script>
```

The goal here is to make development process easier and avoid monolith files.

**`storage.js` — Persistence layer**  
This file should only deal with saving and loading data from `localStorage`.

Think of it as:

> “The place where data is stored when the user closes the tab.”

**`ui.js` — View layer**  
This file should be responsible for:

- Creating HTML strings for different views (list, details, form)
    
- Deciding _how things look in HTML_, not how they behave
    

**`app.js` — Application logic**  
This file should handle:

- Your main data (`tasks`)
    
- Navigation (routing)
    
- User interactions (clicks, form submissions, etc.)
    
- Calling functions from `storage.js` and `ui.js`
    

### 💡 Guidance (not rules)

Before writing code, think about:

- Why separating concerns (storage vs UI vs app logic) might be useful
    
- What would happen if all your code was in one giant file
    

> [!NOTE]
> If you have too much problems with multiple javascript files, you can implement everything inside the app.js file

---

## ✅ **STEP 1.1 — window.UI guide**

This guide is an example for window.UI implementation. It's not necessary to utilize this in your solution and the code examples are provided as an additional help, since the window.UI concept might be too advanced at this point.


## How to use `window.UI` (quick instructions)

### 1) Define `window.UI` in `ui.js`

- Create an object on `window` that contains your UI rendering functions.
    
- Each function should **return an HTML string**.
    

Example pattern (illustrative):

```js
window.UI = {
  tasksView: function (data) { /* return HTML string */ },
  detailsView: function (data) { /* return HTML string */ },
  formView: function (data) { /* return HTML string */ }
};
```

---

### 2) Load `ui.js` before `app.js` in `index.html`

Make sure script order is:

1. `storage.js`
    
2. `ui.js`
    
3. `app.js`
    

```html
<script src="scripts/storage.js"></script>
<script src="scripts/ui.js"></script>
<script src="scripts/app.js"></script>
```

---

### 3) Call `UI` functions from `app.js`

Use `UI.<functionName>(...)` to get HTML, then render it into your app container.

Example pattern (illustrative):

```js
function showTasks() {
  const html = UI.tasksView({ tasks: tasks });
  document.getElementById("app").innerHTML = html;
}
```

---

### 4) Keep responsibilities separate

- `ui.js` / `window.UI` should **only build HTML** (no saving, no routing, no state updates).
    
- `app.js` should:
    
    - manage `tasks`
        
    - handle events (click/submit)
        
    - handle routing
        
    - call `UI` functions to render
        

---

### 5) Test quickly in the console

Open DevTools and run:

- `UI` (should show an object)
    
- `typeof UI.tasksView` (should be `"function"`)
    

If `UI` is `undefined`, check your script order and file paths.


---

# ✅ **STEP 2 — Define Your Data Model (State)**

### 🎯 Goal

Decide what information your app needs to function.

Your app should have a **single main data structure** that represents all tasks.

You should decide:

- What properties a task needs
    
- How you will uniquely identify each task
    

At minimum, each task should include:

- A unique identifier (`id`)
    
- A title
    
- A completed status (`true` or `false`)
    
- One additional piece of information (choose one):
    
    - Priority (low/medium/high)
        
    - Due date
        
    - Category/tag
        
- (Optional but recommended) A timestamp of when the task was created
    

### 💡 Things to think about

- Why is it useful to have a unique `id` instead of relying on list position?
    
- Why might storing everything in one array be helpful?
    
- Why should you avoid storing important data directly in the DOM?
    

_(No strict code required here — just a design decision.)_

---

# ✅ **STEP 3 — Persistence with LocalStorage (storage.js)**

### 🎯 Goal

Make sure tasks persist even if the page is closed or refreshed.

In `storage.js`, you should implement two core responsibilities:

1. **Load tasks from localStorage when the app starts**
    
2. **Save tasks to localStorage whenever they change**
    

### What this file should conceptually do:

- Read a JSON string from `localStorage`
    
- Convert it back into a JavaScript array
    
- Handle cases where no data exists yet
    
- Save updated data back to `localStorage`
    

### Hint

Think in terms of:

> “I need one function that takes my tasks and saves them,  
> and another that retrieves them when the app starts.”

---

# ✅ **STEP 4 — Basic Page Layout (HTML + CSS)**

### 🎯 Goal

Create a simple but clear user interface structure.

Your `index.html` should include at least:

- A **header** with:
    
    - App title
        
    - Navigation links:
        
        - “Tasks”
            
        - “Add Task”
            
- A **main content area** where views will be injected dynamically
    
- A **footer** with a short description or tip
    

Your `main.css` should:

- Make the layout readable on both mobile and desktop
    
- Use **Flexbox or Grid**
    
- Provide basic visual hierarchy (spacing, headings, buttons, etc.)
    
- Clearly distinguish completed tasks (e.g., strikethrough or faded text)
    
- Include visible focus styles for keyboard users
    

### 💡 Design hints

Ask yourself:

- How should the page feel on a phone vs. a laptop?
    
- How can you make buttons clearly look clickable?
    
- How can you visually separate different sections?
    

---

# ✅ **STEP 5 — Understanding Hash-Based Navigation**

### 🎯 Goal

Learn how navigation works without a server.

Your app will use **hash-based routing**, meaning the part of the URL after `#` determines what view is shown.

Examples of routes you should support:

```
#/tasks        → List of tasks
#/add          → Add task form
#/tasks/123    → Details of task with id 123
#/edit/123     → Edit form for task 123
```

### Conceptual steps you must implement:

1. Read `window.location.hash`
    
2. Decide which view should be displayed
    
3. Render that view inside `<main id="app">`
    
4. Re-run this logic whenever the hash changes
    

### 💡 Key idea to understand

The browser does **not** reload the page when the hash changes.  
Your JavaScript decides what to show.

Before coding, think about:

- How you could extract an `id` from `#/tasks/123`
    
- What should happen if someone types an unknown route?
    

---

# ✅ **STEP 6 — Rendering Views (ui.js)**

### 🎯 Goal

Separate UI generation from application logic.

In `ui.js`, you should create functions that **return HTML as strings** for:

- Task list view
    
- Task details view
    
- Add/Edit form view
    

Each function should:

- Take relevant data as input (e.g., tasks array, single task, error message)
    
- Return a string of HTML representing that view
    

### What each view should contain

#### Tasks List View

Must include:

- A list of tasks
    
- A filter option (All / Active / Completed)
    
- For each task:
    
    - Checkbox to toggle completion
        
    - Delete button
        
    - Link to details view
        

If there are no tasks:

- Show a clear “empty state” message
    

#### Add/Edit Form View

Must include:

- Input for title (required)
    
- One additional field (priority/due date/category)
    
- A submit button
    
- A place to show validation errors
    

#### Task Details View

Must include:

- Task title
    
- Completion status
    
- Extra field (priority/due date/category)
    
- Buttons/links to:
    
    - Toggle completion
        
    - Edit task
        
    - Delete task
        
    - Go back to list
        

### 💡 Important design principle

Your UI functions should:

- NOT modify data
    
- NOT handle navigation
    
- ONLY generate HTML based on input
    

---

# ✅ **STEP 7 — Connecting State to UI (app.js)**

### 🎯 Goal

Use your data (`tasks`) to render the correct view.

In `app.js`, you should:

- Keep your main `tasks` array
    
- Call your UI functions from `ui.js`
    
- Insert the returned HTML into `<main id="app">`
    

Every time the user does something that changes data (add, edit, delete, toggle), you should:

1. Update `tasks`
    
2. Save to `localStorage`
    
3. Re-render the appropriate view
    

### 💡 Key idea

Think in this cycle:

> **State → Render → User action → Update state → Render again**

---

# ✅ **STEP 8 — Handling User Input (Forms and Clicks)**

### 🎯 Goal

Respond to user actions efficiently.

You should use **event delegation** instead of attaching listeners to every button.

Conceptually:

- Add one click listener to `document`
    
- Inside that listener, check what was clicked (using `data-action` attributes)
    
- Perform the appropriate action (toggle, delete, etc.)
    

Similarly:

- Add one submit listener for your form
    
- Validate input before modifying data
    

### What must be handled

You must support:

- Adding a new task
    
- Editing an existing task
    
- Toggling completion
    
- Deleting a task (with confirmation)
    

### 💡 Why this matters

This mirrors how real applications manage many interactive elements efficiently.

---

# ✅ **STEP 9 — Navigation + Active Links**

### 🎯 Goal

Make your navigation feel like a real app.

Your navigation bar should:

- Always be visible
    
- Highlight the active view (e.g., underline or different color)
    

When the user clicks “Tasks” or “Add Task”:

- The hash should change
    
- The correct view should render
    
