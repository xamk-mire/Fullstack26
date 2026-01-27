
# Exercise 2 (Follow-up on assignment 1):  Task Tracker in **React + TypeScript + Vite** (with BrowserRouter)

## Overall goal

In this exercise, you will rebuild your Task Tracker application using a modern frontend stack:

- **React** — component-based UI and declarative rendering
    
- **TypeScript** — static typing for safer and more maintainable code
    
- **Vite** — modern development server and build tooling
    
- **React Router (BrowserRouter)** — real URL-based navigation
    

The purpose of this exercise is to help you directly compare:

> Manual DOM + hash routing (vanilla JS)  
> vs.  
> Component-based UI + state-driven rendering + real routing (React)

As you work, pay attention to what becomes easier, what becomes more structured, and how React changes the way you think about UI updates and application state.

---
# Step 1 — Create the project (Vite + React + TypeScript)

### Goal of this step

Set up a clean, modern React + TypeScript project that you will build on for the rest of the exercise. By the end of this step, you should have:

- A running development server
    
- A basic React app rendered in the browser
    
- A project structure created by Vite
    
- Confidence that your environment is working
    

This step is about **tooling and environment**, not application logic yet.

---

### What you need before starting

Make sure you have:

- **Node.js** installed (recommended: LTS version)
    
- A terminal (Command Prompt, PowerShell, Terminal, etc.)
    
- A code editor (VS Code recommended)
    

You can check Node.js with:

```bash
node -v
npm -v
```

If these commands fail, install Node.js before continuing.

---

### Create the Vite project

In your terminal, run:

```bash
npm create vite@latest task-tracker-react
```

When prompted:

- Select **React**
    
- Select **TypeScript**
    

Then:

```bash
cd task-tracker-react
npm install
```

---

### Start the development server

Run:

```bash
npm run dev
```

You should see output similar to:

```
Local:   http://localhost:5173/
```

Open that URL in your browser.

---

### What you should see

In the browser, you should see:

- A default Vite + React page
    
- Some React logos or placeholder content
    
- A page that updates automatically when you save files
    

This confirms that:

- React is working
    
- TypeScript is compiling
    
- Vite dev server is running
    

---

### Explore the generated project (important for understanding)

Take a few minutes to look at these files:

- `src/main.tsx`
    
    - Entry point where React is attached to the DOM
        
- `src/App.tsx`
    
    - The main root component
        
- `index.html`
    
    - The single HTML file used by Vite
        
- `src/` folder
    
    - Where almost all your app code will live
        

### Why this matters

Understanding this structure helps you see:

- React apps usually have **one HTML file**
    
- The UI is created almost entirely with JavaScript/TypeScript + JSX
    
- You are no longer manually writing lots of static HTML for each view
    

---

### Make a small test change (recommended)

To confirm hot reload works:

1. Open `src/App.tsx`
    
2. Change some visible text (for example, the main heading)
    
3. Save the file
    

You should see the browser update **automatically** without refreshing.

---

### Common problems and how to fix them

**Problem:** `npm create vite@latest` not found  
→ Update npm or use `npx create-vite`

**Problem:** Page is blank or shows errors  
→ Check the terminal for TypeScript or build errors

**Problem:** Port already in use  
→ Vite will usually suggest another port automatically

---

### When to move on

Only continue to Step 2 when:

- The dev server runs without errors
    
- You can open the app in the browser
    
- Hot reload works when you edit a file
    

At this point, your development environment is ready, and you can start building the actual Task Tracker features in React.

---

# Step 2 — Enable routing with BrowserRouter (React Router)

### Goal of this step

Add real, URL-based navigation to your React application so that you can build a multi-view app with clean URLs like:

- `/tasks`
    
- `/tasks/123`
    
- `/tasks/new`
    

By the end of this step, you should have:

- React Router installed
    
- Your app wrapped in `BrowserRouter`
    
- A basic routing setup ready for defining pages
    

This replaces the manual hash-based routing you used in the vanilla JavaScript version.

---

### Why routing is important in React apps

In your vanilla app, you manually:

- read `location.hash`
    
- parsed strings to extract IDs
    
- re-rendered views manually
    

With React Router:

- the library listens to URL changes for you
    
- your components automatically re-render when the route changes
    
- route parameters (like `:id`) are provided as values, not strings you must parse yourself
    

This makes navigation:

- more reliable
    
- easier to read
    
- easier to maintain as your app grows
    

---

### Install React Router

In your project folder, run:

```bash
npm install react-router-dom
```

This adds React Router as a dependency to your project.

---

### Wrap your app with BrowserRouter

Open `src/main.tsx`.

Your file already renders `<App />`.  
Update it so that `<App />` is wrapped in `<BrowserRouter>`.

### Example (conceptual structure)

```tsx
import { BrowserRouter } from "react-router-dom";

<BrowserRouter>
  <App />
</BrowserRouter>
```

### What this example includes

- A top-level router component
    
- Enables routing for the entire app tree
    

### Why this is required

If you forget this step:

- `<Routes>`, `<Route>`, `<Link>`, and `<NavLink>` will not work
    
- You may see runtime errors in the browser
    

---

### Understand how BrowserRouter works (conceptually)

`BrowserRouter` uses the browser’s **History API** to:

- Change the URL without reloading the page
    
- Keep UI in sync with the address bar
    
- Allow users to bookmark and refresh pages
    

This is different from:

- hash routing (`#/tasks`)
    
- and closer to how real production React apps work
    

---

### Create a placeholder routing structure (optional but helpful)

At this stage, you don’t need full pages yet.  
However, it can be helpful to set up a minimal routing skeleton in `App.tsx`.

### Example (temporary structure for testing)

```tsx
import { Routes, Route } from "react-router-dom";

<Routes>
  <Route path="/tasks" element={<div>Tasks page (coming soon)</div>} />
</Routes>
```

### What this example shows

- How routes are declared
    
- How a path maps to a React element
    

### Why this helps

This lets you:

- verify routing works
    
- see content change when you go to `/tasks`
    
- confirm setup before building real pages
    

---

### Test routing in the browser

After saving files:

1. Go to: `http://localhost:5173/tasks`
    
2. You should see your placeholder content
    
3. Try changing the URL manually in the address bar
    

If this works, routing is correctly installed.

---

### Common beginner issues

**Problem:** Blank page after adding Routes  
→ Check for:

- Missing `<BrowserRouter>`
    
- Incorrect imports from `react-router-dom`
    

**Problem:** TypeScript errors about JSX  
→ Restart dev server to ensure types are picked up

**Problem:** Page reloads when clicking links  
→ Make sure you are using `<Link>` or `<NavLink>`, not `<a href>`

---

### How this connects to later steps

In the next steps, you will:

- Add a shared Layout component
    
- Define all required routes
    
- Connect routes to real page components
    
- Use route parameters (`:id`) for task details and editing
    

This step lays the foundation for all navigation in your React app.

---

# Step 3 — Define the Task domain model (TypeScript)

### Goal of this step

Create a clear, shared definition of what a “Task” is in your application. By the end of this step, you should have:

- A `Task` type (or interface) defined in one place
    
- A consistent set of fields used across pages and components
    
- A foundation for safer, clearer code throughout the project
    

This is your app’s **data contract**: every part of the app should agree on what a Task contains.

---

### Why we do this early

In small projects, it’s tempting to just “start coding” and store tasks as random objects. But that quickly causes problems like:

- different parts of the app expecting different fields
    
- accidental typos (`compeleted` vs `completed`)
    
- unclear assumptions (is `priority` required? what values are allowed?)
    

TypeScript helps by enforcing consistency. Defining the model early makes the rest of the app easier to build.

---

### Where to put the model

Create a folder that holds “domain” or “types”:

```
src/domain/task.ts
```

Why this location makes sense:

- It’s not UI code
    
- It’s not routing code
    
- It’s a shared definition used everywhere
    

---

### What fields should a Task have?

Your Task should match the original vanilla assignment:

**Required fields:**

- `id` (unique string)
    
- `title` (string)
    
- `completed` (boolean)
    
- one extra field (choose one): priority / dueDate / category
    
    - for this exercise, we’ll use **priority** as the example
        

**Recommended (helps later):**

- `createdAt` (number timestamp)
    
- `updatedAt` (optional number timestamp)
    

---

### Example (Task model shape)

This is an example of a clear Task type. You can adapt fields if your need to.

```ts
// src/domain/task.ts (example)
export type Priority = "low" | "medium" | "high";

export interface Task {
  id: string;
  title: string;
  completed: boolean;
  priority: Priority;
  createdAt: number;
  updatedAt?: number;
}
```

#### What this example includes

- `Priority` is a union type: it restricts possible values
    
- `Task` is an interface used in state, storage, and UI
    

#### Why it matters

If you accidentally write:

```ts
priority: "urgent"
```

TypeScript will warn you immediately.

---

### How you will use this model later

You will import `Task` in many places:

- **state**: `useState<Task[]>(...)`
    
- **storage**: `loadTasks(): Task[]`
    
- **components**: props like `TaskItem({ task }: { task: Task })`
    
- **pages**: finding tasks, displaying task details, editing tasks safely
    

This is one of the biggest advantages of TypeScript in React apps: your code becomes more predictable.

---

### Create a sample task object (for your own sanity check)

A helpful practice is to create a small example object _only for your understanding_ (you don’t need to keep it).

A task might look like:

- `id`: `"abc123"`
    
- `title`: `"Buy milk"`
    
- `completed`: `false`
    
- `priority`: `"medium"`
    
- `createdAt`: `Date.now()`
    

This helps you visualize the data you’ll display in the UI.

---

### Common beginner mistakes (and how to avoid them)

#### Mistake 1: Using `any`

If you write:

```ts
const [tasks, setTasks] = useState<any[]>([]);
```

You lose the benefits of TypeScript. Try to always use:

```ts
useState<Task[]>(...)
```

#### Mistake 2: Using `number` for `id`

You _can_ use numbers, but strings work better for routing because URL params are strings. Also, UUIDs are strings.

#### Mistake 3: Making fields inconsistent

For example:

- some tasks have `priority`
    
- some tasks don’t
    

Decide whether `priority` is always present. If it is optional, TypeScript should reflect that:

```ts
priority?: Priority;
```

But for beginners, it’s usually easier to keep it required and always default to `"medium"`.

---

### When to move on

You are ready to continue when:

- You have a `src/domain/task.ts` file
    
- Your `Task` and `Priority` definitions compile without errors
    
- You understand what each field is used for
    

This model will be used in Step 4 (storage) and Step 7 (React state).

---
# Step 4 — Create a LocalStorage helper (persistence layer)

### Goal of this step

In the vanilla version, you had a `storage.js` file that handled saving/loading tasks. In React, you still want the same idea:

- **UI components should not deal with JSON parsing**
    
- **Pages should not repeat `localStorage.getItem(...)` everywhere**
    
- The app should have **one clear place** that owns persistence rules (key name, parsing, error handling)
    

By the end of this step, you will have a small module that provides two functions:

- `loadTasks()` → returns `Task[]`
    
- `saveTasks(tasks)` → saves tasks
    

✅ In this step you _may copy_ the code below.

---

### Why we use a separate storage file

If you keep storage logic inside pages/components, you often end up with:

- repeated code
    
- inconsistent keys
    
- bugs when parsing fails
    
- mixing “data layer” logic with UI logic
    

A small storage file gives you a clean separation:

> **storage module** = how data is stored  
> **React components** = how data is displayed and edited

---

### Where to put it

Create:

```
src/storage/tasksStorage.ts
```

---

### Copyable example implementation

Paste the following code into `src/storage/tasksStorage.ts`:

```ts
import type { Task } from "../domain/task";

const KEY = "taskTracker.tasks.v1";

/**
 * Loads tasks from localStorage.
 * - Returns an empty array if nothing is stored.
 * - Returns an empty array if stored data is broken/corrupted.
 */
export function loadTasks(): Task[] {
  const raw = localStorage.getItem(KEY);
  if (!raw) return [];

  try {
    const parsed: unknown = JSON.parse(raw);

    // Basic safety check: ensure it's an array
    if (!Array.isArray(parsed)) return [];

    // We trust the data shape here for simplicity.
    // In bigger apps, you would validate each object more strictly.
    return parsed as Task[];
  } catch {
    // If JSON.parse fails, recover gracefully
    return [];
  }
}

/**
 * Saves tasks to localStorage as JSON.
 */
export function saveTasks(tasks: Task[]): void {
  localStorage.setItem(KEY, JSON.stringify(tasks));
}

/**
 * Optional helper: clears stored tasks (useful during development).
 */
export function clearTasks(): void {
  localStorage.removeItem(KEY);
}
```

---

### What this code is doing (beginner-friendly explanation)

### `KEY`

```ts
const KEY = "taskTracker.tasks.v1";
```

This is the “name” under which tasks are saved in the browser. Using a version (`v1`) is useful because:

- If you change your data model later, you can move to `v2` without breaking old data.
    

---

### `loadTasks()`

This function:

1. Reads a string from localStorage
    
2. If nothing exists → return `[]`
    
3. Parses JSON
    
4. If parsing fails or the result isn’t an array → return `[]`
    
5. Otherwise returns `Task[]`
    

Key idea: **the app should never crash because storage is broken.**  
Instead, it should recover gracefully.

---

### `saveTasks(tasks)`

This function:

1. Converts tasks array to JSON string using `JSON.stringify`
    
2. Writes it into localStorage under the key
    

---

### `clearTasks()`

Not required, but helpful during development if you want to reset quickly.

---

### Quick test (recommended)

After implementing Step 4:

1. Open your app in the browser
    
2. Open DevTools → Application (or Storage) → Local Storage
    
3. Find the key:
    
    - `taskTracker.tasks.v1`
        

If you later implement saving correctly, you should see it appear and update.

---

### Common problems and how to fix them

### Problem: “tasks don’t persist”

Most often, saving isn’t being called yet.  
That will be handled in Step 7 when you connect `saveTasks` to React state with `useEffect`.

### Problem: “JSON.parse error”

This is exactly why we have a `try/catch`. If parsing fails, `loadTasks()` returns `[]`.

### Problem: “TypeScript says localStorage is not defined”

That usually happens if you try to run code in a non-browser environment. In Vite + React in the browser, it should be fine.

---

### When to move on

You are ready for the next step when:

- You created `tasksStorage.ts`
    
- It exports `loadTasks` and `saveTasks`
    
- Your project compiles without errors
    

---
# Step 5 — Create a shared `Layout` component (Header + Navigation + Outlet)

### Goal of this step

In your vanilla app, `index.html` had a header and navigation that stayed visible while the content inside `#app` changed. In React with React Router, you’ll recreate that same idea using a **Layout component**.

By the end of this step, you will have:

- A **Layout component** that is always visible
    
- A navigation bar with links to:
    
    - **Tasks** (`/tasks`)
        
    - **Add Task** (`/tasks/new`)
        
- A main content area where different pages render
    
- Active navigation styling (so students can see which page they are on)
    

This step is important because it teaches a very common React pattern:

> Use a shared layout so you don’t repeat the same header/nav on every page.

---

### Where to create the file

Create:

```
src/components/Layout.tsx
```

---

## What a Layout component does (conceptually)

A Layout component usually has three parts:

1. **Header** (branding/title)
    
2. **Navigation** (links between views)
    
3. **Outlet** (placeholder for the current route’s page)
    

The key part is **`<Outlet />`**:

- It is provided by React Router
    
- It is the “slot” where the page content will appear
    
- When the URL changes, React Router renders the correct page _inside the Outlet_
    

So the Layout stays the same, and only the page content changes.

---

## Example Layout code (copyable)

You can copy this and then adjust styling/text as you like:

```tsx
import { NavLink, Outlet } from "react-router-dom";
import "../styles/app.css";

export default function Layout() {
  return (
    <div className="container">
      <header className="header">
        <div>
          <h1 className="title">Task Tracker</h1>
          <p className="subtitle">React + TypeScript + Vite</p>
        </div>

        <nav className="nav" aria-label="Primary navigation">
          <NavLink
            to="/tasks"
            className={({ isActive }) => (isActive ? "navLink active" : "navLink")}
          >
            Tasks
          </NavLink>

          <NavLink
            to="/tasks/new"
            className={({ isActive }) => (isActive ? "navLink active" : "navLink")}
          >
            Add Task
          </NavLink>
        </nav>
      </header>

      <main className="main">
        <Outlet />
      </main>
    </div>
  );
}
```

---

## What this example includes (and why)

### ✅ `NavLink` instead of `Link`

`NavLink` is like `Link`, but it can tell you when the link is currently active.

That lets you style active navigation links so the user knows where they are.

In the example, we use:

```tsx
className={({ isActive }) => (isActive ? "navLink active" : "navLink")}
```

This means:

- if the current URL matches the link → add `active` class
    
- otherwise → normal link class
    

---

### ✅ `aria-label` on `<nav>`

This helps accessibility. Screen readers can identify navigation areas more clearly.

---

### ✅ `<Outlet />`

This is the most important piece:

- Without it, your pages won’t show up when routing changes
    
- It acts like the “page content slot”
    

---

## Add minimal CSS so it looks decent (recommended)

Create or update:

```
src/styles/app.css
```

Here is a simple beginner-friendly style set (you can also use your own styles e.g. from the assignment 1):

```css
.container {
  max-width: 900px;
  margin: 0 auto;
  padding: 16px;
  font-family: system-ui, Arial, sans-serif;
}

.header {
  display: flex;
  gap: 16px;
  align-items: center;
  justify-content: space-between;
  flex-wrap: wrap;
  padding: 12px 0;
  border-bottom: 1px solid #ddd;
}

.title {
  margin: 0;
  font-size: 24px;
}

.subtitle {
  margin: 4px 0 0;
  color: #666;
}

.nav {
  display: flex;
  gap: 10px;
}

.navLink {
  text-decoration: none;
  padding: 8px 12px;
  border: 1px solid #ddd;
  border-radius: 8px;
  color: #333;
}

.navLink.active {
  border-color: #333;
  font-weight: 600;
}

.main {
  padding: 16px 0;
}
```

---

## How to verify your Layout works

You’ll know it’s correct when:

✅ The header and navigation appear on the page  
✅ When you later add routes, the page content changes _inside the main area_  
✅ The active link styling updates when you navigate

---

## Common beginner mistakes

### Mistake 1: Forgetting `<Outlet />`

Result: Your header shows, but pages never appear.

Fix: Make sure `<Outlet />` exists inside the Layout component.

---

### Mistake 2: Using `<a href="/tasks">`

This causes full page reloads.

Fix: Use `<Link>` or `<NavLink>` from React Router.

---

### Mistake 3: Active styling never shows

Usually caused by:

- using `Link` instead of `NavLink`, or
    
- not applying classes correctly
    

Fix: Use `NavLink` + the `isActive` callback pattern shown above.

---

## When to move on

You’re ready to continue when:

- `Layout.tsx` exists
    
- It contains header + nav + `<Outlet />`
    
- The app compiles
    
- You understand that the Outlet is where pages will render
    

---

# Step 6 — Define your routes in `App.tsx` (Pages + URLs)

### Goal of this step

Connect **URLs** to **React page components** using React Router.

By the end of this step, you should have:

- A routing “map” that defines where each page lives
    
- A Layout that stays visible (header/nav)
    
- Pages that render inside the Layout via `<Outlet />`
    
- A default redirect from `/` to `/tasks`
    
- A “Not Found” route for unknown URLs
    

This step replaces your vanilla router logic and teaches a core React Router concept:

> Routes are declared in code, and the router decides what to render based on the current URL.

---

### Before you start: create placeholder pages

In `src/pages/`, create these files (they can be very simple for now):

- `TasksListPage.tsx`
    
- `TaskDetailsPage.tsx`
    
- `TaskFormPage.tsx`
    
- `NotFoundPage.tsx`
    

### Why placeholders help

It’s easier to build routing step-by-step if each route points to something that renders immediately, even if it’s temporary text.

### Example placeholder page (beginner-friendly)

```tsx
export default function TasksListPage() {
  return <h2>Tasks List Page</h2>;
}
```

You can do the same idea for the other pages.

---

### Where routing lives

Routing is usually defined in `src/App.tsx`.

Your `App` component becomes the place where you describe:

- which paths exist
    
- which component renders for each path
    

---

### Route requirements (from the assignment)

You must implement:

- `/tasks`
    
- `/tasks/new`
    
- `/tasks/:id`
    
- `/tasks/:id/edit`
    

---

### Nested routes: why we use them

You created `Layout.tsx` in Step 5. Now you want it to appear on **every page**.

React Router supports this with **nested routes**:

- The parent route renders the Layout
    
- Child routes render pages inside the Layout’s `<Outlet />`
    

So conceptually:

- Parent route: `/` → Layout
    
- Children: `/tasks`, `/tasks/new`, etc. → pages
    

---

### Example route structure (copyable)

This is a working route structure you can copy and then adjust.

```tsx
import { Navigate, Route, Routes } from "react-router-dom";
import Layout from "./components/Layout";

import TasksListPage from "./pages/TasksListPage";
import TaskDetailsPage from "./pages/TaskDetailsPage";
import TaskFormPage from "./pages/TaskFormPage";
import NotFoundPage from "./pages/NotFoundPage";

export default function App() {
  return (
    <Routes>
      {/* Parent route: always show Layout */}
      <Route path="/" element={<Layout />}>
        {/* When user goes to "/", redirect to "/tasks" */}
        <Route index element={<Navigate to="/tasks" replace />} />

        {/* List page */}
        <Route path="tasks" element={<TasksListPage />} />

        {/* Create page */}
        <Route path="tasks/new" element={<TaskFormPage mode="create" />} />

        {/* Details page (":id" is a route parameter) */}
        <Route path="tasks/:id" element={<TaskDetailsPage />} />

        {/* Edit page */}
        <Route path="tasks/:id/edit" element={<TaskFormPage mode="edit" />} />

        {/* Fallback for unknown routes */}
        <Route path="*" element={<NotFoundPage />} />
      </Route>
    </Routes>
  );
}
```

---

### Explanation of what this routing code is doing

#### 1) `<Routes>` and `<Route>`

- `<Routes>` is the container for all route definitions
    
- `<Route>` maps a path → element
    

---

#### 2) `path="/" element={<Layout />}`

This means:

- whenever the user is on any matching child route
    
- React Router renders `Layout`
    
- and then renders the matching child route inside the Layout’s `<Outlet />`
    

---

#### 3) `index element={<Navigate ... />}`

This is the default route **inside** the parent route.

When users go to:

- `/`
    

they automatically get redirected to:

- `/tasks`
    

The `replace` option means it doesn’t clutter the back button history.

---

#### 4) Route parameters: `tasks/:id`

This is how you represent “details for a specific task”.

For example:

- `/tasks/abc123`
    
- `/tasks/9f2a...`
    

React Router will treat `abc123` as a value for `id`.

Later, in your details page, you’ll read it like:

- `const { id } = useParams();`
    

---

#### 5) Reusing a page component for create and edit

You’re using:

```tsx
<TaskFormPage mode="create" />
<TaskFormPage mode="edit" />
```

This is a clean beginner-friendly design because:

- The UI is very similar
    
- Only the behavior changes
    
    - create → make a new task
        
    - edit → update an existing task
        

In Step 10 you’ll build the logic inside TaskFormPage.

---

### How to verify routing works (important!)

After you add these routes and placeholder pages:

1. Start the dev server (`npm run dev`)
    
2. In the browser, visit:
    
    - `/tasks`
        
    - `/tasks/new`
        
    - `/tasks/anything`
        
    - `/tasks/anything/edit`
        
    - `/something-random`
        

✅ Expected behavior:

- You see your placeholder headings
    
- Layout header/nav remains visible
    
- Unknown paths show NotFoundPage
    
- `/` redirects to `/tasks`
    

---

### Common beginner mistakes

### Mistake 1: Pages don’t show up

Often caused by forgetting `<Outlet />` in `Layout`.

Fix: Ensure Layout includes `<Outlet />`.

---

### Mistake 2: Wrong paths (extra slashes)

If your child route is written like:

```tsx
<Route path="/tasks" ... />
```

inside a parent route, it becomes an absolute path (different behavior).

Fix: For nested routes, use relative paths like:

```tsx
<Route path="tasks" ... />
```

---

### Mistake 3: `TaskFormPage` gives a TypeScript error about `mode`

If you pass props like `mode="create"`, your component must accept them.

Fix: In `TaskFormPage.tsx`, define props:

```ts
type Props = { mode: "create" | "edit" };
```

---

## When to move on

You are ready to continue when:

- All required routes are defined
    
- Layout stays visible while pages change
    
- `/` redirects to `/tasks`
    
- Unknown URLs show a Not Found page
    

---

# Step 7 — Manage tasks as React state (state in `App` + props)

### Goal of this step

In the vanilla version, you had one `tasks` array and you updated it manually, then re-rendered.

In React, the equivalent idea is:

- keep `tasks` in **React state**
    
- update tasks using `setTasks(...)`
    
- save tasks to localStorage automatically when tasks change
    
- pass tasks + update functions down to pages via props
    

By the end of Step 7, you should have:

- `tasks` state stored in `App.tsx` (single source of truth)
    
- persistence wired (load once, save on change)
    
- a clear plan for passing data to pages
    

This is beginner-friendly and very explicit way of handling state in react apps, and more will be introduced later.

---

### 1) Store tasks in `App.tsx`

### What you do

In `App.tsx`, create state for tasks. You will:

1. Load tasks from localStorage once at startup
    
2. Store them in `useState`
    
3. Save tasks whenever the state changes (with `useEffect`)
    

### Example (copyable)

This example shows the key pattern you should implement:

```tsx
import { useEffect, useState } from "react";
import type { Task } from "./domain/task";
import { loadTasks, saveTasks } from "./storage/tasksStorage";

export default function App() {
  const [tasks, setTasks] = useState<Task[]>(() => loadTasks());

  useEffect(() => {
    saveTasks(tasks);
  }, [tasks]);

  // Routes will go here (Step 6)
  return (
    // ...
    <div>TODO: Routes</div>
  );
}
```

### Explanation (beginner-friendly)

- `useState(() => loadTasks())` runs `loadTasks()` only once during the initial render.
    
- `useEffect(..., [tasks])` runs every time `tasks` changes, so your app automatically saves.
    

This is the React version of “save + rerender” from the vanilla app—but React handles rerendering for you.

---

### 2) Create “task actions” in `App.tsx`

### What you do

Define functions in `App.tsx` that update tasks. Pages will call these functions via props.

You will likely need actions like:

- addTask
    
- updateTask
    
- deleteTask
    
- toggleTaskCompleted
    

### Examples (copyable patterns)

#### Add task

```tsx
function addTask(newTask: Task) {
  setTasks(prev => [...prev, newTask]);
}
```

#### Delete task

```tsx
function deleteTask(id: string) {
  setTasks(prev => prev.filter(t => t.id !== id));
}
```

#### Toggle completion

```tsx
function toggleCompleted(id: string) {
  setTasks(prev =>
    prev.map(t => (t.id === id ? { ...t, completed: !t.completed } : t))
  );
}
```

#### Update task fields (title/priority)

```tsx
function updateTask(id: string, patch: Partial<Task>) {
  setTasks(prev =>
    prev.map(t => (t.id === id ? { ...t, ...patch, updatedAt: Date.now() } : t))
  );
}
```

### Why these patterns matter

React expects state updates to be **immutable**:

- don’t change the old array/object
    
- create a new array/object with the changes
    

That’s why we use:

- `map` for updating a single item
    
- `filter` for removing
    
- `[...prev, item]` for adding
    

---

### 3) Pass state and actions to pages via props

### What you do

Update your routes (from Step 6) so that pages receive:

- `tasks` (data)
    
- action callbacks (functions)
    

### Example (conceptual)

```tsx
<Route
  path="tasks"
  element={<TasksListPage tasks={tasks} onToggle={toggleCompleted} onDelete={deleteTask} />}
/>
```

### What this example shows

- Pages don’t own state
    
- Pages can still update state by calling props like `onToggle(id)`
    
- `App` stays the “single source of truth”
    

---

### 4) Define prop types in pages (TypeScript)

### What you do

In each page, define props that match what `App` provides.

### Example (TasksListPage props)

```tsx
import type { Task } from "../domain/task";

type Props = {
  tasks: Task[];
  onToggle: (id: string) => void;
  onDelete: (id: string) => void;
};

export default function TasksListPage({ tasks, onToggle, onDelete }: Props) {
  // render tasks here
  return <div>{tasks.length}</div>;
}
```

### Why this matters

- Your pages become reusable and testable
    
- TypeScript helps you keep props consistent
    

---

## How to verify Step 7 is correct

After implementing Step 7, you should be able to:

✅ Render the number of tasks in any page using props  
✅ Update tasks from a page by calling callback props  
✅ Refresh the browser and see tasks persist (after you implement add/edit later)

A simple check:

- temporarily display `tasks.length` somewhere
    
- confirm it updates when tasks change
    

---

## Common beginner mistakes

### Mistake 1: Mutating state directly

Bad pattern:

```tsx
tasks.push(newTask);
setTasks(tasks);
```

This can lead to React not re-rendering correctly.

Correct pattern:

```tsx
setTasks(prev => [...prev, newTask]);
```

### Mistake 2: Saving tasks in every page

If your `useEffect` in `App` saves tasks, you should not need to call `saveTasks()` elsewhere.

### Mistake 3: Passing the wrong props to routes

TypeScript will help here—if the props don’t match, you’ll get a compile error.

---

## When to move on

You’re ready for Step 8 when:

- `tasks` is stored in `App.tsx`
    
- `useEffect` saves tasks automatically
    
- You have action functions (add/update/delete/toggle)
    
- You are passing tasks/actions into pages via props
    

---

# Step 8 — Build the **Tasks List page** (`/tasks`)

### Goal of this step

Create the main page of the app: a list of tasks that users can interact with.

By the end of this step, your `/tasks` page should:

- Display all tasks from props
    
- Show an **empty state** if there are no tasks
    
- Let the user **toggle completion** (checkbox)
    
- Let the user **delete** a task (button + optional confirmation)
    
- Let the user navigate to:
    
    - **task details** (`/tasks/:id`)
        
    - **edit task** (`/tasks/:id/edit`)
        
- Provide a link to **add a new task** (`/tasks/new`)
    

This page will help you practice the most common React UI pattern:

> Take data (tasks) → render UI using `.map()` → trigger actions using props callbacks.

---

### 1) Create the page file and define props clearly

### What you do

Create:

```
src/pages/TasksListPage.tsx
```

This page should receive data and callbacks from `App.tsx` via props.

### Example (props definition)

```tsx
import type { Task } from "../domain/task";

type Props = {
  tasks: Task[];
  onToggle: (id: string) => void;
  onDelete: (id: string) => void;
};
```

### Why this matters

- The page itself does **not** own the tasks state
    
- It only displays tasks and calls `onToggle`/`onDelete` when the user interacts
    
- This keeps `App.tsx` as the single source of truth (Step 7)
    

---

### 2) Render a heading and a “Add Task” link

### What you do

Add a header section so users can see where they are and create tasks easily.

### Example (basic structure)

```tsx
import { Link } from "react-router-dom";

<h2>Tasks</h2>
<Link to="/tasks/new">+ Add Task</Link>
```

### Why this matters

In React Router, you should use:

- `<Link to="...">` instead of `<a href="...">`
    

Because:

- `<Link>` navigates without reloading the page
    
- `<a>` does a full page refresh, which is not what we want in SPAs
    

---

### 3) Show an empty state when there are no tasks

### What you do

If `tasks.length === 0`, show a friendly message and a clear call-to-action.

### Example (empty state pattern)

```tsx
if (tasks.length === 0) {
  return (
    <section>
      <h2>Tasks</h2>
      <p>No tasks yet. Create your first task!</p>
      <Link to="/tasks/new">+ Add Task</Link>
    </section>
  );
}
```

### Why this matters

An empty list with no message feels broken.  
A good empty state makes the app feel complete and easier to use.

---

### 4) Render tasks using `.map()` and `key`

### What you do

If tasks exist, render them as a list.

### Example (core React list rendering)

```tsx
<ul>
  {tasks.map(task => (
    <li key={task.id}>
      {task.title}
    </li>
  ))}
</ul>
```

### What this example shows

- `.map()` creates one `<li>` per task
    
- `key={task.id}` is required so React can track items reliably
    

### Why `key` matters

React uses keys to understand what changed between renders.  
If keys are missing (or unstable), you can get strange UI behavior.

---

### 5) Add completion toggle (checkbox)

### What you do

Show a checkbox that reflects task completion and triggers `onToggle(task.id)`.

### Example (checkbox wiring)

```tsx
<input
  type="checkbox"
  checked={task.completed}
  onChange={() => onToggle(task.id)}
/>
```

### Why this matters

- `checked={task.completed}` makes it controlled by React state
    
- `onChange` triggers an update
    
- When state updates, React automatically re-renders, and the checkbox updates
    

---

### 6) Add navigation links for details and edit

### What you do

Make task titles clickable and add an edit link.

### Example (details + edit links)

```tsx
<Link to={`/tasks/${task.id}`}>{task.title}</Link>
<Link to={`/tasks/${task.id}/edit`}>Edit</Link>
```

### Why this matters

This is how multi-view apps work in React:

- pages are just routes
    
- links navigate to those routes
    

---

### 7) Add delete button with confirmation (recommended)

### What you do

Provide a delete button that calls `onDelete(task.id)`.  
Optionally confirm before deleting.

### Example (button + confirm)

```tsx
<button
  type="button"
  onClick={() => {
    const ok = confirm(`Delete "${task.title}"?`);
    if (ok) onDelete(task.id);
  }}
>
  Delete
</button>
```

### Why this matters

Deleting is destructive. Confirmation reduces accidental loss.

---

### 8) Put it all together 

This example shows how your page might be structured. You can copy it and then adjust styling and layout.

```tsx
import { Link } from "react-router-dom";
import type { Task } from "../domain/task";

type Props = {
  tasks: Task[];
  onToggle: (id: string) => void;
  onDelete: (id: string) => void;
};

export default function TasksListPage({ tasks, onToggle, onDelete }: Props) {
  return (
    <section>
      <header style={{ display: "flex", justifyContent: "space-between", alignItems: "center" }}>
        <h2>Tasks</h2>
        <Link to="/tasks/new">+ Add Task</Link>
      </header>

      {tasks.length === 0 ? (
        <p>No tasks yet. Add one to get started!</p>
      ) : (
        <ul>
          {tasks.map(task => (
            <li key={task.id} style={{ display: "flex", gap: "12px", alignItems: "center" }}>
              <input
                type="checkbox"
                checked={task.completed}
                onChange={() => onToggle(task.id)}
                aria-label={`Toggle completion for ${task.title}`}
              />

              <Link to={`/tasks/${task.id}`}>
                {task.title}
              </Link>

              <span style={{ marginLeft: "auto" }}>
                <Link to={`/tasks/${task.id}/edit`}>Edit</Link>
                {" "}
                <button
                  type="button"
                  onClick={() => {
                    const ok = confirm(`Delete "${task.title}"?`);
                    if (ok) onDelete(task.id);
                  }}
                >
                  Delete
                </button>
              </span>
            </li>
          ))}
        </ul>
      )}
    </section>
  );
}
```

### What this example includes

- A clear header with “Add Task”
    
- Empty state handling
    
- Rendering tasks with `.map()` and `key`
    
- Checkbox toggle wired to props
    
- Links for details/edit
    
- Delete action with confirmation
    

---

## How to connect this page in `App.tsx`

In Step 7 you created actions and tasks state in `App`. Now you pass them into this page through your route:

### Example (route element props)

```tsx
<Route
  path="tasks"
  element={<TasksListPage tasks={tasks} onToggle={toggleCompleted} onDelete={deleteTask} />}
/>
```

---

## How to verify Step 8 is correct

✅ When you visit `/tasks`, you should see:

- “Tasks” heading
    
- “Add Task” link
    

✅ If there are no tasks:

- you see an empty message
    

✅ If tasks exist:

- you see each task
    
- toggling a checkbox updates completion immediately
    
- clicking “Delete” removes a task
    
- clicking a title opens details (once Step 9 is implemented)
    

---

## Common beginner mistakes

### Mistake 1: Using `<a href>`

This reloads the page. Use `<Link>`.

### Mistake 2: Forgetting `key`

React will warn you in the console.

### Mistake 3: Calling the function immediately

Bad:

```tsx
onChange={onToggle(task.id)}
```

This runs immediately during render.

Correct:

```tsx
onChange={() => onToggle(task.id)}
```

---

## When to move on

You’re ready for Step 9 when:

- `/tasks` renders correctly
    
- toggle and delete actions work
    
- links exist for details/edit
    
- the page renders an empty state cleanly
    

---
# Step 9 — Build the **Task Details page** (`/tasks/:id`)

### Goal of this step

Create a page that shows **one specific task** based on the URL. For example:

- `/tasks/abc123` shows details for the task with id `abc123`
    

By the end of this step, your Task Details page should:

- Read the `id` from the route (`/tasks/:id`)
    
- Find the matching task from the `tasks` array (passed via props)
    
- Show task information (title, status, priority, etc.)
    
- Provide actions:
    
    - Toggle completion
        
    - Delete task
        
    - Navigate to edit page
        
    - Navigate back to task list
        
- Handle missing/invalid ids with a friendly “Task not found” view
    

---

## 1) Create the page file and define its props

### What you do

Create:

```
src/pages/TaskDetailsPage.tsx
```

Because you are using state in `App`, this page should receive tasks and callbacks as props.

### Example (props shape)

```tsx
import type { Task } from "../domain/task";

type Props = {
  tasks: Task[];
  onToggle: (id: string) => void;
  onDelete: (id: string) => void;
};
```

### Why this matters

- The details page doesn’t store tasks itself
    
- It uses props to read tasks and trigger updates
    
- This keeps `App` as the single source of truth
    

---

## 2) Read the `id` from the URL using `useParams()`

### What you do

React Router provides `useParams()` to access route parameters like `:id`.

### Example (reading params)

```tsx
import { useParams } from "react-router-dom";

const { id } = useParams();
```

### What this example includes

- `id` will be a string (or `undefined` if missing)
    
- It matches whatever appears in the URL after `/tasks/`
    

### Why this matters

This replaces manual string parsing from your vanilla router.

---

## 3) Find the correct task from the tasks array

### What you do

Use `Array.find(...)` to locate the task by id.

### Example (finding the task)

```tsx
const task = tasks.find(t => t.id === id);
```

### What this example includes

- `task` will be `undefined` if no match exists
    

### Why this matters

Your UI must handle invalid URLs gracefully.

---

## 4) Handle the “task not found” case

### What you do

If the task does not exist, show a friendly message and a link back.

### Example (not found UI)

```tsx
if (!task) {
  return (
    <section>
      <h2>Task not found</h2>
      <p>The task you are trying to view does not exist.</p>
      <Link to="/tasks">Back to tasks</Link>
    </section>
  );
}
```

### Why this matters

Users can type URLs manually or click outdated links. A good app doesn’t crash.

---

## 5) Display the task details clearly

### What you should show

At minimum:

- Title
    
- Completed status
    
- Priority (or your chosen extra field)
    

### Example (simple task display)

```tsx
<h2>{task.title}</h2>
<p>Status: {task.completed ? "Completed" : "Active"}</p>
<p>Priority: {task.priority}</p>
```

### Why this matters

This is the purpose of the page: users should be able to _confirm what task they are viewing_.

---

## 6) Provide navigation actions (Back + Edit)

### What you do

Add links for common actions:

- Back to list: `/tasks`
    
- Edit page: `/tasks/:id/edit`
    

### Example (links)

```tsx
<Link to="/tasks">← Back</Link>
<Link to={`/tasks/${task.id}/edit`}>Edit</Link>
```

### Why this matters

In multi-view apps, users should never feel “stuck” on a page.

---

## 7) Add behavior actions (Toggle + Delete)

### Toggle completion

Call the `onToggle(task.id)` prop.

### Example

```tsx
<button type="button" onClick={() => onToggle(task.id)}>
  {task.completed ? "Mark as active" : "Mark as completed"}
</button>
```

### Delete task (recommended with confirmation)

Call `onDelete(task.id)` prop.

### Example

```tsx
<button
  type="button"
  onClick={() => {
    const ok = confirm(`Delete "${task.title}"?`);
    if (ok) onDelete(task.id);
  }}
>
  Delete
</button>
```

### Important note (navigation after delete)

If the user deletes a task while viewing its details, that task no longer exists. In many apps, the expected behavior is to redirect back to `/tasks`.

You can handle this in two ways:

- Redirect inside the details page (using `useNavigate`)
    
- Or handle it in the delete logic in `App`
    

A beginner-friendly approach is: use `useNavigate()` in this page.

---

## 8) (Recommended) Navigate back to `/tasks` after deleting

### What you do

Use `useNavigate()` to move the user away after deletion.

### Example (concept)

```tsx
import { useNavigate } from "react-router-dom";

const navigate = useNavigate();

const ok = confirm("...");
if (ok) {
  onDelete(task.id);
  navigate("/tasks");
}
```

### Why this matters

Otherwise the page will still try to display a task that was just deleted.

---

## 9) A complete example (you can copy and adapt)

This is a reasonable “reference implementation” for the page. You can copy it and then improve layout/styling.

```tsx
import { Link, useNavigate, useParams } from "react-router-dom";
import type { Task } from "../domain/task";

type Props = {
  tasks: Task[];
  onToggle: (id: string) => void;
  onDelete: (id: string) => void;
};

export default function TaskDetailsPage({ tasks, onToggle, onDelete }: Props) {
  const { id } = useParams();
  const navigate = useNavigate();

  const task = tasks.find(t => t.id === id);

  if (!task) {
    return (
      <section>
        <h2>Task not found</h2>
        <p>The task you are trying to view does not exist.</p>
        <Link to="/tasks">Back to tasks</Link>
      </section>
    );
  }

  return (
    <section>
      <Link to="/tasks">← Back</Link>

      <h2>{task.title}</h2>
      <p>Status: {task.completed ? "Completed" : "Active"}</p>
      <p>Priority: {task.priority}</p>

      <div style={{ display: "flex", gap: "10px", marginTop: "12px", flexWrap: "wrap" }}>
        <button type="button" onClick={() => onToggle(task.id)}>
          {task.completed ? "Mark as active" : "Mark as completed"}
        </button>

        <Link to={`/tasks/${task.id}/edit`}>Edit</Link>

        <button
          type="button"
          onClick={() => {
            const ok = confirm(`Delete "${task.title}"?`);
            if (!ok) return;
            onDelete(task.id);
            navigate("/tasks");
          }}
        >
          Delete
        </button>
      </div>
    </section>
  );
}
```

---

## 10) Connect this page in `App.tsx`

You must pass props when creating the route element.

### Example (route element with props)

```tsx
<Route
  path="tasks/:id"
  element={<TaskDetailsPage tasks={tasks} onToggle={toggleCompleted} onDelete={deleteTask} />}
/>
```

---

## How to verify Step 9 works

✅ Visit `/tasks/:id` by clicking a task title from the list  
✅ You see the correct task title and details  
✅ Toggle updates completion state  
✅ Delete removes the task and navigates back to `/tasks`  
✅ Invalid ID (e.g. `/tasks/does-not-exist`) shows “Task not found”

---

## Common beginner mistakes

- Forgetting `useParams()` → you won’t know which task to display
    
- Forgetting “not found” handling → your app might crash or show blank data
    
- Deleting without navigation → user stays on a page that references a removed task
    
- Using `<a href>` instead of `<Link>` → full page reloads
    

---

# Step 10 — Build the **Task Form page** (Create + Edit + Validation)

### Goal of this step

Create a form page that works in two modes:

- **Create mode** (`/tasks/new`) → user adds a new task
    
- **Edit mode** (`/tasks/:id/edit`) → user edits an existing task
    

By the end of this step, your form page should:

- Show input fields for:
    
    - Title (required)
        
    - Priority (low/medium/high)
        
- Validate that title is not empty (show an error message)
    
- In **create mode**: add a new task and navigate to `/tasks`
    
- In **edit mode**: pre-fill inputs with current task values, update task, and navigate back to `/tasks/:id`
    
- Handle invalid edit URLs (`/tasks/does-not-exist/edit`) with “Task not found”
    

This step teaches a core React concept:

> In React, forms are usually built with **controlled inputs**, where input values are stored in component state.

---

## 1) Create the page file and define props

### What you do

Create:

```
src/pages/TaskFormPage.tsx
```

This page needs:

- the full task list (to find the task in edit mode)
    
- an “add task” function
    
- an “update task” function
    

### Example (props shape)

```tsx
import type { Task } from "../domain/task";

type Props = {
  mode: "create" | "edit";
  tasks: Task[];
  onAdd: (task: Task) => void;
  onUpdate: (id: string, patch: Partial<Task>) => void;
};
```

### Why this matters

- The page doesn’t own the global tasks state
    
- It calls `onAdd` / `onUpdate` to modify state in `App.tsx`
    

---

## 2) Read the `id` from the route in edit mode

### What you do

In edit mode, your URL contains `:id`. Use `useParams()` to get it.

### Example

```tsx
import { useParams } from "react-router-dom";
const { id } = useParams();
```

### Key idea

- In create mode, you typically don’t need `id`
    
- In edit mode, `id` is required to find the task
    

---

## 3) Decide the initial form values

### What you do

You need initial values for your controlled inputs:

- Create mode → empty/default values
    
- Edit mode → pre-fill from the existing task
    

### Example (finding the task to edit)

```tsx
const taskToEdit = mode === "edit"
  ? tasks.find(t => t.id === id)
  : undefined;
```

### If edit task is missing

If `mode === "edit"` and `taskToEdit` is undefined:

- show “Task not found”
    
- provide a link back to `/tasks`
    

---

## 4) Create controlled state for form fields + error message

### What you do

Store the form values in component state so inputs stay in sync with React.

You’ll typically need:

- `title`
    
- `priority`
    
- `error` (string)
    

### Example (controlled state setup)

```tsx
const [title, setTitle] = useState("");
const [priority, setPriority] = useState<"low" | "medium" | "high">("medium");
const [error, setError] = useState("");
```

### Important: pre-filling values in edit mode

A beginner-friendly approach is:

- initialize state using `taskToEdit` if it exists
    

One common pattern is to set initial values when the component loads.

---

## 5) Pre-fill the form in edit mode (recommended approach)

### What you do

When in edit mode and you have a task, you want:

- title input to show existing title
    
- priority select to show existing priority
    

A beginner-friendly way is using `useEffect`:

### Example (pre-fill effect)

```tsx
useEffect(() => {
  if (mode === "edit" && taskToEdit) {
    setTitle(taskToEdit.title);
    setPriority(taskToEdit.priority);
  }
}, [mode, taskToEdit]);
```

### Why this matters

- If a user navigates directly to `/tasks/:id/edit`, the form should be pre-filled.
    
- It also makes your form predictable and user-friendly.
    

---

## 6) Build the form UI (inputs + labels)

### What you do

Create the actual form with labels and controlled inputs.

### Example (title input)

```tsx
<label>
  Title *
  <input
    value={title}
    onChange={(e) => setTitle(e.target.value)}
  />
</label>
```

### Example (priority select)

```tsx
<label>
  Priority
  <select
    value={priority}
    onChange={(e) => setPriority(e.target.value as "low" | "medium" | "high")}
  >
    <option value="low">Low</option>
    <option value="medium">Medium</option>
    <option value="high">High</option>
  </select>
</label>
```

### Why this matters

This is the standard React form pattern:

- input value comes from state
    
- input changes update state
    
- state updates re-render UI
    

---

## 7) Validate form input (title required)

### What you do

When the user submits:

- trim title
    
- if empty → show error and stop
    

### Example (validation)

```tsx
const trimmed = title.trim();
if (trimmed.length === 0) {
  setError("Title is required.");
  return;
}
setError("");
```

### Why this matters

Validation is required by the assignment and improves usability.

---

## 8) Handle submit (create vs edit)

### What you do

On form submit:

## Create mode

- create a new Task object
    
- call `onAdd(newTask)`
    
- navigate to `/tasks`
    

## Edit mode

- call `onUpdate(id, { title, priority })`
    
- navigate to `/tasks/:id`
    

You’ll use `useNavigate()` for navigation.

### Example (navigate setup)

```tsx
import { useNavigate } from "react-router-dom";
const navigate = useNavigate();
```

---

## 9) Build the “create task object” correctly

### What a new task needs

For consistency with your earlier model, new tasks should include:

- unique `id`
    
- title
    
- completed = false
    
- priority
    
- createdAt = Date.now()
    

### Example (generating id)

```tsx
const newId =
  crypto.randomUUID ? crypto.randomUUID() : String(Date.now());
```

### Example (new task shape)

```tsx
const newTask: Task = {
  id: newId,
  title: trimmed,
  completed: false,
  priority,
  createdAt: Date.now(),
};
```

---

## 10) A complete TaskFormPage example (copy + adapt)

This page uses everything from steps above. You can copy it and then improve styling/layout.

```tsx
import { Link, useNavigate, useParams } from "react-router-dom";
import { useEffect, useState } from "react";
import type { Task } from "../domain/task";

type Props = {
  mode: "create" | "edit";
  tasks: Task[];
  onAdd: (task: Task) => void;
  onUpdate: (id: string, patch: Partial<Task>) => void;
};

export default function TaskFormPage({ mode, tasks, onAdd, onUpdate }: Props) {
  const { id } = useParams();
  const navigate = useNavigate();

  const taskToEdit = mode === "edit" ? tasks.find(t => t.id === id) : undefined;

  const [title, setTitle] = useState("");
  const [priority, setPriority] = useState<"low" | "medium" | "high">("medium");
  const [error, setError] = useState("");

  useEffect(() => {
    if (mode === "edit" && taskToEdit) {
      setTitle(taskToEdit.title);
      setPriority(taskToEdit.priority);
    }
  }, [mode, taskToEdit]);

  // If edit mode but task is missing → show not found
  if (mode === "edit" && !taskToEdit) {
    return (
      <section>
        <h2>Task not found</h2>
        <p>Cannot edit because the task does not exist.</p>
        <Link to="/tasks">Back to tasks</Link>
      </section>
    );
  }

  function handleSubmit(e: React.FormEvent) {
    e.preventDefault();

    const trimmed = title.trim();
    if (trimmed.length === 0) {
      setError("Title is required.");
      return;
    }
    setError("");

    if (mode === "create") {
      const newId = crypto.randomUUID ? crypto.randomUUID() : String(Date.now());

      const newTask: Task = {
        id: newId,
        title: trimmed,
        completed: false,
        priority,
        createdAt: Date.now(),
      };

      onAdd(newTask);
      navigate("/tasks");
      return;
    }

    // edit mode
    onUpdate(taskToEdit!.id, { title: trimmed, priority });
    navigate(`/tasks/${taskToEdit!.id}`);
  }

  return (
    <section>
      <h2>{mode === "create" ? "Add Task" : "Edit Task"}</h2>

      <form onSubmit={handleSubmit}>
        <label>
          Title *
          <input value={title} onChange={(e) => setTitle(e.target.value)} />
        </label>

        <label>
          Priority
          <select
            value={priority}
            onChange={(e) => setPriority(e.target.value as "low" | "medium" | "high")}
          >
            <option value="low">Low</option>
            <option value="medium">Medium</option>
            <option value="high">High</option>
          </select>
        </label>

        <div style={{ marginTop: "12px", display: "flex", gap: "10px" }}>
          <button type="submit">{mode === "create" ? "Add" : "Save"}</button>
          <Link to={mode === "create" ? "/tasks" : `/tasks/${taskToEdit!.id}`}>Cancel</Link>
        </div>

        {error && <p style={{ color: "crimson" }}>{error}</p>}
      </form>
    </section>
  );
}
```

---

## 11) Connect TaskFormPage routes in `App.tsx` with props

Now you must pass `tasks`, `onAdd`, and `onUpdate` into the route elements.

### Example (route elements with props)

```tsx
<Route
  path="tasks/new"
  element={<TaskFormPage mode="create" tasks={tasks} onAdd={addTask} onUpdate={updateTask} />}
/>

<Route
  path="tasks/:id/edit"
  element={<TaskFormPage mode="edit" tasks={tasks} onAdd={addTask} onUpdate={updateTask} />}
/>
```

---

## How to verify Step 10 works

✅ Create mode:

- go to `/tasks/new`
    
- submit empty title → shows error
    
- submit valid title → task appears in list
    

✅ Edit mode:

- click edit on a task
    
- form is pre-filled
    
- change title/priority and save
    
- details page shows updated values
    

✅ Invalid edit URL:

- go to `/tasks/not-real/edit`
    
- you see “Task not found” and can go back
    

---

## Common beginner mistakes

- Forgetting to use controlled inputs (`value` + `onChange`)
    
- Not trimming title before validation (`" "` should be invalid)
    
- Not navigating after create/edit (user stays on the form)
    
- Not handling “task not found” in edit mode
    

---
