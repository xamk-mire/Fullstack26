# JavaScript Fundamentals: Make the Study Planner Interactive

Now that you have:

* **HTML** for structure
* **CSS** for styling

…it’s time to add **behavior** with JavaScript.

In this build-along, you’ll make the “Add task” form actually work:

✅ Add tasks to the list
✅ Mark tasks as done (checkbox)
✅ Remove tasks
✅ Store tasks in the browser (localStorage) so they persist after refresh
✅ Practice core JS skills: variables, functions, arrays, objects, events, DOM manipulation

You’ll still use **no frameworks**. This is exactly the kind of manual work React later simplifies.

---

## 0) Small HTML updates (so JS can “grab” elements easily)

Open your `index.html` and make these small edits.

### A. Give the task list an `id`

Find your tasks list inside the “Today” section and update it like this:

```html
<ul id="taskList">
  <li>Read lecture notes</li>
  <li>Complete HTML exercises</li>
  <li>Commit changes to Git</li>
</ul>
```

### B. Give the form an `id`

Find your “Add a task” form and update:

```html
<form id="taskForm">
```

### C. Add IDs to inputs

Update the inputs so they match these IDs (keep `name` too):

```html
<input id="title" name="title" type="text" placeholder="e.g., Finish exercises" required>
<input id="due" name="due" type="date">
<input id="important" type="checkbox" name="important">
```

### D. Include the script at the bottom of `<body>`

Right before `</body>`, add:

```html
<script src="script.js"></script>
```

Create a new file next to your HTML: `script.js`

---

## 1) Goal: Use a data model (array of objects)

In real apps you don’t just “add HTML”. You store data, then render it.

We’ll store tasks like this:

```js
{
  id: "some-unique-id",
  title: "Complete JavaScript exercises",
  due: "2026-01-13",
  important: true,
  done: false
}
```

That’s:

* an **object** representing one task
* an **array** to store many tasks

---

## 2) Set up DOM references + state

Put this in `script.js`:

```js
// ---- DOM references ----
const taskForm = document.querySelector("#taskForm");
const taskList = document.querySelector("#taskList");

const titleInput = document.querySelector("#title");
const dueInput = document.querySelector("#due");
const importantInput = document.querySelector("#important");

// ---- State (our tasks live here) ----
let tasks = [];
```

### What’s happening?

* `querySelector` finds HTML elements by CSS selector
* `tasks` is our in-memory list of tasks

---

## 3) Load and save tasks with localStorage

Add this below your state:

```js
const STORAGE_KEY = "studyPlannerTasks";

function saveTasks() {
  localStorage.setItem(STORAGE_KEY, JSON.stringify(tasks));
}

function loadTasks() {
  const raw = localStorage.getItem(STORAGE_KEY);
  if (!raw) return [];
  try {
    return JSON.parse(raw);
  } catch {
    return [];
  }
}
```

### Explanation

* `localStorage` stores strings
* `JSON.stringify` converts objects/arrays → string
* `JSON.parse` converts string → objects/arrays

---

## 4) Render tasks into the list (DOM building)

Replace the static tasks in the HTML later if you want—JS will overwrite them anyway.

Add this renderer:

```js
function renderTasks() {
  // Clear existing content
  taskList.innerHTML = "";

  for (const task of tasks) {
    const li = document.createElement("li");

    // Build a row container
    const row = document.createElement("div");
    row.className = "task-row";

    // Done checkbox
    const doneCheckbox = document.createElement("input");
    doneCheckbox.type = "checkbox";
    doneCheckbox.checked = task.done;
    doneCheckbox.addEventListener("change", () => {
      toggleDone(task.id);
    });

    // Title text
    const titleSpan = document.createElement("span");
    titleSpan.textContent = task.title;

    if (task.done) titleSpan.classList.add("task-done");
    if (task.important) titleSpan.classList.add("task-important");

    // Due date text
    const dueSpan = document.createElement("span");
    dueSpan.className = "task-due";
    dueSpan.textContent = task.due ? `Due: ${task.due}` : "";

    // Delete button
    const deleteButton = document.createElement("button");
    deleteButton.type = "button";
    deleteButton.textContent = "Remove";
    deleteButton.addEventListener("click", () => {
      removeTask(task.id);
    });

    row.append(doneCheckbox, titleSpan, dueSpan, deleteButton);
    li.append(row);
    taskList.append(li);
  }
}
```

### Key JS concepts here

* Creating elements with `document.createElement`
* Setting properties (`textContent`, `checked`, `type`)
* Adding event listeners (`addEventListener`)
* Updating the DOM by appending children

---

## 5) Add task logic (form submit)

Add these helper functions:

```js
function createTask({ title, due, important }) {
  return {
    id: crypto.randomUUID ? crypto.randomUUID() : String(Date.now()),
    title,
    due,
    important,
    done: false,
  };
}

function addTask(task) {
  tasks.push(task);
  saveTasks();
  renderTasks();
}
```

Now handle form submit:

```js
taskForm.addEventListener("submit", (event) => {
  event.preventDefault(); // stop the page from reloading

  const title = titleInput.value.trim();
  const due = dueInput.value;
  const important = importantInput.checked;

  if (!title) return; // required also prevents, but this is good practice

  addTask(createTask({ title, due, important }));

  // Reset form
  taskForm.reset();
  titleInput.focus();
});
```

### Why `preventDefault()`?

Forms normally submit and reload the page. We want to handle it in JavaScript instead.

---

## 6) Toggle done + remove task

Add:

```js
function toggleDone(taskId) {
  tasks = tasks.map((t) =>
    t.id === taskId ? { ...t, done: !t.done } : t
  );
  saveTasks();
  renderTasks();
}

function removeTask(taskId) {
  tasks = tasks.filter((t) => t.id !== taskId);
  saveTasks();
  renderTasks();
}
```

### Concepts used

* `.map()` to update one item immutably
* `.filter()` to remove one item

---

## 7) Initialize the app on page load

At the bottom of `script.js`, add:

```js
tasks = loadTasks();
renderTasks();
```

Now refresh your page:

* Add tasks
* Mark them done
* Remove them
* Refresh again: they should still be there

---

## 8) (Optional but recommended) Small CSS add-ons for the task list

Add this to your `styles.css` to make task rows look nicer:

```css
.task-row {
  display: flex;
  align-items: center;
  gap: 10px;
}

.task-row button {
  margin-left: auto;
}

.task-done {
  text-decoration: line-through;
  opacity: 0.6;
}

.task-important {
  font-weight: 700;
}

.task-due {
  font-size: 0.9rem;
  color: #555;
}
```

---

# Full `script.js` (Complete)

If you want one full copy:

```js
const taskForm = document.querySelector("#taskForm");
const taskList = document.querySelector("#taskList");

const titleInput = document.querySelector("#title");
const dueInput = document.querySelector("#due");
const importantInput = document.querySelector("#important");

const STORAGE_KEY = "studyPlannerTasks";

let tasks = [];

function saveTasks() {
  localStorage.setItem(STORAGE_KEY, JSON.stringify(tasks));
}

function loadTasks() {
  const raw = localStorage.getItem(STORAGE_KEY);
  if (!raw) return [];
  try {
    return JSON.parse(raw);
  } catch {
    return [];
  }
}

function createTask({ title, due, important }) {
  return {
    id: crypto.randomUUID ? crypto.randomUUID() : String(Date.now()),
    title,
    due,
    important,
    done: false,
  };
}

function addTask(task) {
  tasks.push(task);
  saveTasks();
  renderTasks();
}

function toggleDone(taskId) {
  tasks = tasks.map((t) =>
    t.id === taskId ? { ...t, done: !t.done } : t
  );
  saveTasks();
  renderTasks();
}

function removeTask(taskId) {
  tasks = tasks.filter((t) => t.id !== taskId);
  saveTasks();
  renderTasks();
}

function renderTasks() {
  taskList.innerHTML = "";

  for (const task of tasks) {
    const li = document.createElement("li");

    const row = document.createElement("div");
    row.className = "task-row";

    const doneCheckbox = document.createElement("input");
    doneCheckbox.type = "checkbox";
    doneCheckbox.checked = task.done;
    doneCheckbox.addEventListener("change", () => toggleDone(task.id));

    const titleSpan = document.createElement("span");
    titleSpan.textContent = task.title;
    if (task.done) titleSpan.classList.add("task-done");
    if (task.important) titleSpan.classList.add("task-important");

    const dueSpan = document.createElement("span");
    dueSpan.className = "task-due";
    dueSpan.textContent = task.due ? `Due: ${task.due}` : "";

    const deleteButton = document.createElement("button");
    deleteButton.type = "button";
    deleteButton.textContent = "Remove";
    deleteButton.addEventListener("click", () => removeTask(task.id));

    row.append(doneCheckbox, titleSpan, dueSpan, deleteButton);
    li.append(row);
    taskList.append(li);
  }
}

taskForm.addEventListener("submit", (event) => {
  event.preventDefault();

  const title = titleInput.value.trim();
  const due = dueInput.value;
  const important = importantInput.checked;

  if (!title) return;

  addTask(createTask({ title, due, important }));

  taskForm.reset();
  titleInput.focus();
});

tasks = loadTasks();
renderTasks();
```

---

## Practical exercises (students can try after build-along)

1. **Clear all tasks**

   * Add a “Clear all” button that deletes everything from storage

2. **Show task count**

   * Add a badge next to “Today” showing the number of tasks

3. **Sort tasks**

   * Important tasks first, then by due date

4. **Validation message**

   * If title is empty, show a message under the form instead of silently returning

5. **Edit task title**

   * Add an “Edit” button that prompts the user and updates the title
