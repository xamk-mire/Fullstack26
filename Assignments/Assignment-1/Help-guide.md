
# Task Tracker —  Help Guide (If You Get Stuck)

This guide is here to provide additional details and help in case you need it.

Try first to come up with your own implementation and use the help guide to compare your own solution or to receive help.


## How to use this guide

- Find the step you’re stuck on.
    
- Read the “What’s the idea?” section first.
    
- Use the snippets to understand the pattern.
    
- Implement it in your own code (rename variables, reorganize functions, change HTML layout).
    

---

## Step 1 — Scripts not loading / “X is not defined”

### What’s the idea?

When you open `index.html` directly and you’re **not using modules**, the browser loads scripts **top to bottom**.  
So anything used later must be defined earlier.

### Minimal HTML shell
```html
<!doctype html>
<html>
  <head>
    <meta charset="utf-8" />
    <meta name="viewport" content="width=device-width,initial-scale=1" />
    <title>Task Tracker</title>
    <link rel="stylesheet" href="styles/main.css" />
  </head>
  <body>
    <header>
      <h1>Task Tracker</h1>
      <nav aria-label="Primary">
        <a href="#/tasks" id="navTasks">Tasks</a>
        <a href="#/add" id="navAdd">Add Task</a>
      </nav>
    </header>

    <main id="app"></main>

    <footer>
      <small>Open index.html directly — no server required.</small>
    </footer>

    <!-- IMPORTANT: order matters -->
    <script src="scripts/storage.js"></script>
    <script src="scripts/ui.js"></script>
    <script src="scripts/app.js"></script>
  </body>
</html>
```

### Quick checks

Open DevTools Console and type:

- `window.Storage`
    
- `window.UI` - you can find example in step 6
    

If either is `undefined`, that file didn’t load or didn’t attach to `window`.

### Common causes

- Typo in file path (`script/` vs `scripts/`)
    
- Wrong script order
    
- `ui.js` didn’t assign `window.UI = ...`

---

## Step 2 — Defining a “task object” that won’t cause problems later

### What’s the idea?

Your app has **one source of truth**: the tasks array in `app.js`.  

The DOM is just a _view_ of that state.

Pick a consistent data shape now so your UI and routing are predictable.

### Example task shape

```js
// Think of this as the *shape*, not required exact fields
const exampleTask = {
  id: "1700000000000",     // unique
  title: "Read chapter 3",  // required
  completed: false,         // required
  priority: "medium",       // your extra field (or dueDate/category)
  createdAt: 1700000000000  // optional but useful
};
```

### Why an `id` is essential

Routes like `#/tasks/:id` require stable identity.  
If you use array index as id, the “id” changes when you delete/reorder tasks.

### Two simple ways to generate ids

- `String(Date.now())` (easy)
    
- `crypto.randomUUID()` (best, if available)
---

## Step 3 — LocalStorage that doesn’t randomly crash

### What’s the idea?

LocalStorage only stores **strings**, so we store tasks as JSON.

**Load once at startup** → **save after every change**.

**Process**

1. Convert tasks array → JSON string using `JSON.stringify`
    
2. Save into localStorage under a key
    
3. On load, read the string and parse with `JSON.parse`
    
4. Handle missing data or broken JSON
    

### Example `storage.js` pattern

```js
(function () {
  const KEY = "taskTracker.tasks.v1";

  function load() {
    const raw = localStorage.getItem(KEY);
    if (!raw) return []; // nothing saved yet

    try {
      const parsed = JSON.parse(raw);
      return Array.isArray(parsed) ? parsed : [];
    } catch (err) {
      // if parsing fails, return empty instead of crashing the app
      return [];
    }
  }

  function save(tasks) {
    localStorage.setItem(KEY, JSON.stringify(tasks));
  }

  window.Storage = { load, save };
})();
```

### How the process works

- `save(tasks)` converts the array into a JSON string and stores it.
    
- `load()` retrieves the string and converts it back into an array.
    
- `try/catch` prevents your app from breaking if saved data is malformed.
    

### Debug tip

After adding tasks, refresh and run:

- `localStorage.getItem("taskTracker.tasks.v1")`
    

You should see a JSON string.

---
## Step 4 — HTML structure + CSS: “What should my base layout look like?”

### What’s the idea?

Your `index.html` is the **shell**. Views render inside `#app`.

You want:

- a consistent header/nav
    
- a single render target (`#app`)
    
- CSS that makes tasks readable and actions obvious
    

### Suggested `index.html` shell (example)

```html
<header class="siteHeader">
  <h1>Task Tracker</h1>
  <nav aria-label="Primary">
    <a href="#/tasks" id="navTasks">Tasks</a>
    <a href="#/add" id="navAdd">Add Task</a>
  </nav>
</header>

<main id="app"></main>

<footer class="siteFooter">
  <small>Tip: Open index.html directly (no server).</small>
</footer>
```

### A minimal “task row” HTML pattern you might render (example)

```html
<li class="taskRow" data-task-id="1700000000000">
  <label>
    <input type="checkbox" data-action="toggle" />
    <span class="taskTitle">Buy milk</span>
  </label>

  <a href="#/tasks/1700000000000">Details</a>
  <a href="#/edit/1700000000000">Edit</a>
  <button type="button" data-action="delete">Delete</button>
</li>
```

### CSS ideas (example)

```css
.taskRow { display: flex; gap: 12px; align-items: center; }
.taskRow.completed .taskTitle { text-decoration: line-through; opacity: 0.6; }

nav a.active { text-decoration: underline; font-weight: 600; }

button { cursor: pointer; }
:focus-visible { outline: 3px solid #7aa2ff; outline-offset: 2px; }
```

### How this helps later

- `data-task-id` lets buttons “know” which task they belong to.
    
- `data-action` lets you use event delegation (one handler for many buttons).
    
- `.completed` is a simple, clear state style.

---
## Step 5 — Routing + view functions: “How do `showTasks`, `showDetails`, and `showForm` work together?”

This is the section you asked to improve most.

### What’s the idea?

Your router decides which “showX” function to call based on the hash.  
Each `showX` function:

1. finds relevant data from state (`tasks`)
    
2. asks `UI` to generate HTML
    
3. renders HTML into `#app`
    

### Router example (example)

```js
function router() {
  const hash = location.hash || "#/tasks";

  setActiveNav(hash);

  if (hash === "#/tasks") return showTasks();

  if (hash === "#/add") return showForm({ mode: "add" });

  if (hash.startsWith("#/tasks/")) {
    const id = hash.slice("#/tasks/".length);
    return showDetails({ id });
  }

  if (hash.startsWith("#/edit/")) {
    const id = hash.slice("#/edit/".length);
    return showForm({ mode: "edit", id });
  }

  // Unknown route
  return showNotFound(); // or redirect to #/tasks
}
```

### `render()` helper (simple but powerful)

```js
function render(html) {
  document.getElementById("app").innerHTML = html;
}
```

---

### `showTasks()` example (example)

**What it does:**

- passes the whole tasks array into the list view
    
- the UI can decide how to show empty state vs list
    

```js
function showTasks() {
  render(UI.tasksView({ tasks }));
}
```

---

### `showDetails({ id })` example 

**What it does (step-by-step):**

1. Finds the task in state
    
2. If not found, render a friendly message
    
3. Otherwise render the details UI
    

```js
function showDetails({ id }) {
  const task = tasks.find(t => t.id === id);

  if (!task) {
    render(UI.notFoundView({ message: "Task not found.", backHref: "#/tasks" }));
    return;
  }

  render(UI.detailsView({ task }));
}
```

**Why this pattern is “correct”**

- It never assumes the task exists.
    
- It keeps “finding data” in app logic, not in the UI layer.
    
- UI just receives `task` and renders it.
    

---

### `showForm({ mode, id, error })` example 

**What it does (step-by-step):**

- In `add` mode: task is `null`
    
- In `edit` mode: it loads the task and pre-fills the form
    
- If edit id is invalid: show “Task not found”
    

```js
function showForm({ mode, id = null, error = "" }) {
  let task = null;

  if (mode === "edit") {
    task = tasks.find(t => t.id === id);
    if (!task) {
      render(UI.notFoundView({ message: "Cannot edit: task not found.", backHref: "#/tasks" }));
      return;
    }
  }

  render(UI.formView({ mode, task, error }));
}
```

**Why this pattern helps**

- Your submit handler can re-render the same form with an error by calling `showForm(...)` again.
    
- UI can prefill fields when `task` exists.
    

---

### Setup: calling router at the right times

```js
window.addEventListener("DOMContentLoaded", () => {
  tasks = Storage.load();  // load state before first render
  router();
});

window.addEventListener("hashchange", router);
```


---
## Step 6 — UI functions: list, details, form (HTML examples + how data-action works)

### What’s the idea?

UI functions return HTML strings. They don’t update state.

### A minimal tasks view shape (example)

```js
// ui.js (example)
window.UI = {
  tasksView({ tasks }) {
    if (tasks.length === 0) {
      return `
        <section>
          <h2>Tasks</h2>
          <p>No tasks yet — add one!</p>
          <a href="#/add">+ Add Task</a>
        </section>
      `;
    }

    const items = tasks.map(t => `
      <li class="taskRow ${t.completed ? "completed" : ""}" data-task-id="${t.id}">
        <label>
          <input type="checkbox" data-action="toggle" ${t.completed ? "checked" : ""} />
          <span class="taskTitle">${escapeHtml(t.title)}</span>
        </label>
        <a href="#/tasks/${t.id}">Details</a>
        <a href="#/edit/${t.id}">Edit</a>
        <button type="button" data-action="delete">Delete</button>
      </li>
    `).join("");

    return `
      <section>
        <h2>Tasks</h2>
        <a href="#/add">+ Add Task</a>
        <ul class="taskList">${items}</ul>
      </section>
    `;
  },

  // ... detailsView, formView, notFoundView ...
};
```

### Important: escaping user input

If students allow task titles, those are user-provided strings. A simple helper prevents HTML injection.

example:

```js
function escapeHtml(str) {
  return String(str)
    .replaceAll("&", "&amp;")
    .replaceAll("<", "&lt;")
    .replaceAll(">", "&gt;")
    .replaceAll('"', "&quot;")
    .replaceAll("'", "&#039;");
}
```

---

### Details view shape (example)

```js
detailsView({ task }) {
  return `
    <section data-task-id="${task.id}">
      <a href="#/tasks">← Back</a>
      <h2>Task details</h2>

      <p><strong>Title:</strong> ${escapeHtml(task.title)}</p>
      <p><strong>Status:</strong> ${task.completed ? "Completed" : "Active"}</p>
      <p><strong>Priority:</strong> ${escapeHtml(task.priority || "—")}</p>

      <div class="actions">
        <button type="button" data-action="toggle">Toggle complete</button>
        <a href="#/edit/${task.id}">Edit</a>
        <button type="button" data-action="delete">Delete</button>
      </div>
    </section>
  `;
}
```

Notice:

- `data-task-id` is present so the same delegated event handler can work here too.
    

---

### Form view shape (example)

```js
formView({ mode, task, error }) {
  const isEdit = mode === "edit";
  const title = task ? task.title : "";
  const priority = task ? task.priority : "medium";

  return `
    <section>
      <a href="${isEdit ? `#/tasks/${task.id}` : "#/tasks"}">← Back</a>
      <h2>${isEdit ? "Edit task" : "Add task"}</h2>

      <form id="taskForm" novalidate>
        <label>
          Title *
          <input name="title" value="${escapeHtml(title)}" />
        </label>

        <label>
          Priority
          <select name="priority">
            <option value="low" ${priority === "low" ? "selected" : ""}>Low</option>
            <option value="medium" ${priority === "medium" ? "selected" : ""}>Medium</option>
            <option value="high" ${priority === "high" ? "selected" : ""}>High</option>
          </select>
        </label>

        <button type="submit">${isEdit ? "Save" : "Add"}</button>

        ${error ? `<p class="errorMsg">${escapeHtml(error)}</p>` : ""}
      </form>
    </section>
  `;
}
```

Key idea:

- In edit mode, fields are prefilled.
    
- `error` is displayed if present.
    

---


## Step 7 — The “commit” pattern (save + rerender) to avoid repeating yourself

### What’s the idea?

A lot of your actions do the same thing afterward:

- save tasks
    
- re-render current view (or navigate)
    

### Example

```js
function commit({ rerender = true } = {}) {
  Storage.save(tasks);
  if (rerender) router();
}
```

When to use:

- Toggle completion → `commit()`
    
- Delete task → `commit()` then navigate if needed
    
- Add/edit often navigates → so you might save then change hash
    

---

## Step 8 — Events: toggle, delete, and form submit (expanded)

### What’s the idea?

Use:

- one delegated click handler for toggle/delete
    
- one submit handler for add/edit
    

### Delegated click handler (works on list and details)

```js
document.addEventListener("click", (e) => {
  const action = e.target.getAttribute("data-action");
  if (!action) return;

  const container = e.target.closest("[data-task-id]");
  const id = container ? container.getAttribute("data-task-id") : null;
  if (!id) return;

  if (action === "toggle") {
    const task = tasks.find(t => t.id === id);
    if (!task) return;
    task.completed = !task.completed;
    commit();
  }

  if (action === "delete") {
    const task = tasks.find(t => t.id === id);
    if (!task) return;

    const ok = confirm(`Delete "${task.title}"?`);
    if (!ok) return;

    tasks = tasks.filter(t => t.id !== id);

    // If you deleted while viewing details, you might want to go back
    if (location.hash.startsWith("#/tasks/")) location.hash = "#/tasks";
    else commit();
  }
});
```

How it works:

- `data-action` tells you what behavior is requested.
    
- `closest("[data-task-id]")` finds the relevant task wrapper.
    
- Same handler works across multiple views.
    

---

### Form submit handler (add + edit)

```js
document.addEventListener("submit", (e) => {
  if (e.target.id !== "taskForm") return;
  e.preventDefault();

  const title = e.target.title.value.trim();
  const priority = e.target.priority.value;

  if (!title) {
    // Re-render the current form view with an error
    if (location.hash.startsWith("#/edit/")) {
      const id = location.hash.slice("#/edit/".length);
      showForm({ mode: "edit", id, error: "Title is required." });
    } else {
      showForm({ mode: "add", error: "Title is required." });
    }
    return;
  }

  if (location.hash.startsWith("#/edit/")) {
    const id = location.hash.slice("#/edit/".length);
    const task = tasks.find(t => t.id === id);
    if (!task) return;

    task.title = title;
    task.priority = priority;
    Storage.save(tasks);
    location.hash = "#/tasks/" + id;
    return;
  }

  // Add mode
  const newTask = {
    id: String(Date.now()),
    title,
    completed: false,
    priority
  };
  tasks.push(newTask);
  Storage.save(tasks);
  location.hash = "#/tasks";
});
```

Why this approach is solid:

- Validation errors re-render the form while keeping the user on the same route.
    
- Edit submits navigate to details, add submits navigate to list.
    

---

## Step 9 — Active nav highlighting (simple and effective)

### What’s the idea?

Give user a clear “where am I?” cue.

### HTML example

Your nav links already exist:

```html
<a href="#/tasks" id="navTasks">Tasks</a>
<a href="#/add" id="navAdd">Add Task</a>
```

### JS example 

```js
function setActiveNav(hash) {
  document.getElementById("navTasks")?.classList.remove("active");
  document.getElementById("navAdd")?.classList.remove("active");

  if (hash.startsWith("#/add") || hash.startsWith("#/edit/")) {
    document.getElementById("navAdd")?.classList.add("active");
  } else {
    document.getElementById("navTasks")?.classList.add("active");
  }
}
```

### CSS example

```css
nav a.active {
  text-decoration: underline;
  font-weight: 600;
}
```


“Back” link patterns:

- Details view usually goes back to `#/tasks`
    
- Edit view “back” can go to `#/tasks/:id`
    
- Add view “back” can go to `#/tasks`
    

This can be hard-coded into the HTML strings your UI renders (as shown in the form/details examples).


---
## Debugging checklist (expanded)

When something doesn’t work, check in this order:

1. **Console errors** (fix the first one)
    
2. Does clicking nav change `location.hash`?
    
3. Does `router()` run on hash changes? (add `console.log("router", location.hash)`)
    
4. Is `tasks` loaded before first render?
    
5. Are your `data-task-id` wrappers present in the rendered HTML?
    
6. Are actions labeled with `data-action`?
    
7. After updates: are you saving and re-rendering (commit/router)?
    
