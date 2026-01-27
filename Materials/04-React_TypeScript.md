# TypeScript & React — From Web Pages to Web Apps

Up to this point, you’ve been working with the web in its most direct form: you wrote HTML that the browser can read, you added CSS to shape what it looks like, and you used JavaScript to make things happen when a user clicks, types, or submits a form.

That’s the core of the web.

But you probably noticed something important: as soon as the page becomes even a little interactive, the code starts to feel fragile. You’re constantly finding elements, updating text, creating nodes, removing nodes, keeping track of what exists, and trying not to forget to re-render something. It works—but it’s easy to get lost.

This is the moment when modern tools start to make sense.

In this module, you’ll learn two of the most widely used ideas in modern frontend development:

- **TypeScript**, which helps you write JavaScript that’s easier to maintain
    
- **React**, which changes how you build interfaces by treating UI as a function of data
    

The goal is not just to “use React”, but to understand what problems it solves—and why the patterns you’ll use in React feel different from vanilla DOM manipulation.

---

# TypeScript Fundamentals: JavaScript With Guardrails 

JavaScript is the language of the web, and one of its biggest strengths is flexibility: you can quickly try ideas, build features, and make things work. But that same flexibility can become a problem as your codebase grows—especially in a full-stack project where many parts depend on each other.

TypeScript is designed to keep JavaScript’s flexibility while adding something crucial:

> **Clarity about data.**  
> TypeScript lets you describe the shape of the values your code uses, and it checks your code before it runs.

In practice, TypeScript helps you catch mistakes earlier, understand your code faster, and refactor more safely.

---

## 1) What TypeScript Is (and What It Isn’t)

### TypeScript is…

- A superset of JavaScript (valid JS is valid TS)
    
- A tool that adds **static type checking**
    
- A developer-time safety system (errors show up in your editor/terminal)
    

### TypeScript is not…

- A different runtime (your browser still runs JavaScript)
    
- A guarantee that your program can never crash
    
- A replacement for good logic, testing, or validation
    

TypeScript checks your code at build time, then compiles it to JavaScript for the browser.

---

## 2) Why Types Matter: Fewer Surprises

Consider this JavaScript code:

```js
function formatTitle(task) {
  return task.title.toUpperCase();
}
```

This works—until `task.title` isn’t a string.

In JavaScript, the failure is delayed until runtime. In TypeScript, you can define what a task looks like:

```ts
type Task = {
  title: string;
};

function formatTitle(task: Task) {
  return task.title.toUpperCase();
}
```

Now TypeScript can warn you if you call `formatTitle` with the wrong kind of value. You’ve moved bugs from “later” to “now”.

---

## 3) Basic Types You’ll Use All the Time

TypeScript starts with familiar primitives:

```ts
let name: string = "Ada";
let age: number = 20;
let isDone: boolean = false;
```

Other useful basics:

```ts
let tags: string[] = ["web", "react"];
let scores: number[] = [10, 20, 30];
```

TypeScript can often infer types:

```ts
let course = "Fullstack"; // inferred as string
```

A useful habit is:

- rely on inference when it’s obvious
    
- write explicit types when it improves readability or safety
    

---

## 4) Object Types: Describing “Shape”

Most application data is objects. TypeScript shines here.

```ts
type Task = {
  id: string;
  title: string;
  done: boolean;
  due?: string; // optional
};
```

### What does `due?: string` mean?

The `?` means “optional”—the property might be missing.

So this is valid:

```ts
const t1: Task = { id: "1", title: "Read notes", done: false };
```

And this is also valid:

```ts
const t2: Task = { id: "2", title: "Finish lab", done: false, due: "2026-01-27" };
```

But this is not:

```ts
const t3: Task = { id: "3", done: false }; // error: missing title
```

---

## 5) Type Aliases vs Interfaces

You’ll commonly see both `type` and `interface` used to describe object shapes.

### Type alias

```ts
type Task = { id: string; title: string; done: boolean };
```

### Interface

```ts
interface Task {
  id: string;
  title: string;
  done: boolean;
}
```

For this course:

- both are fine
    
- choose one style and be consistent within a project
    

Students often find `type` slightly simpler at first. Interfaces are common in many codebases. You’ll encounter both.

---

## 6) Functions: Types for Inputs and Outputs

TypeScript lets you declare:

- parameter types
    
- return type
    

```ts
function add(a: number, b: number): number {
  return a + b;
}
```

If you forget to return a number, TypeScript complains.

### Functions that return nothing

```ts
function logTask(task: Task): void {
  console.log(task.title);
}
```

`void` means “no return value that matters.”

---

## 7) Union Types: “This OR That”

Sometimes a value can be more than one type.

```ts
type Id = string | number;
```

Example:

```ts
function findTask(id: string | number) {
  // ...
}
```

Union types are extremely common with:

- API results
    
- optional fields
    
- loading/error states
    

A classic example:

```ts
type Status = "idle" | "loading" | "success" | "error";
```

This creates a safer set of allowed values than “any string”.

---

## 8) Narrowing: TypeScript Helps You Check Safely

If something could be multiple types, you often need a check.

```ts
function format(value: string | number) {
  if (typeof value === "number") {
    return value.toFixed(2);
  }
  return value.toUpperCase();
}
```

Inside each branch, TypeScript “narrows” the type.

---

## 9) Generics: Types That Work With Many Shapes

You don’t need to master generics immediately, but you’ll see them.

A common example is arrays:

```ts
const tasks: Array<Task> = [];
```

Or when using functions like `useState` in React:

```ts
const [tasks, setTasks] = useState<Task[]>([]);
```

This tells TypeScript exactly what kind of items live inside the state array.

---

## 10) `any`, `unknown`, and “Type Safety Escape Hatches”

### `any`

`any` means “turn off type checking for this value.”

```ts
let data: any = fetchSomething();
```

It can be tempting, but it removes most of TypeScript’s benefits.

### `unknown`

`unknown` is safer: it means “I don’t know what this is yet.”  
You must check before using it:

```ts
let data: unknown = fetchSomething();

if (typeof data === "string") {
  console.log(data.toUpperCase());
}
```

In this course, a good rule is:

- avoid `any` unless you truly have to
    
- prefer `unknown` for untrusted data
    

---

## 11) TypeScript in React: Why It Matters Immediately

React components depend heavily on:

- props
    
- state
    
- event handlers
    

TypeScript helps by ensuring:

- you pass the right props
    
- your state contains the expected data
    
- you don’t accidentally treat a string like a number
    

Example: typed props

```tsx
type TaskItemProps = {
  title: string;
  done: boolean;
};

function TaskItem({ title, done }: TaskItemProps) {
  return <li>{done ? "✅" : "⬜"} {title}</li>;
}
```

If you pass the wrong type, TypeScript catches it right away.

---

## 12) TypeScript is Not a Substitute for Validation

Even with perfect TypeScript, your frontend can still receive “wrong” data from outside sources:

- APIs
    
- user input
    
- databases
    
- third-party services
    

TypeScript protects your code assumptions, but it does not validate runtime data. That’s why later we still validate:

- form inputs
    
- API requests in the backend (FastAPI + schemas)
    
- data coming from the database
    

---

# The SPA Concept: A Web App That Doesn’t Constantly Reload 

When people say “web app”, they usually mean something that behaves more like a desktop or mobile app than a traditional website. It feels smooth. It updates quickly. You can click around without seeing full page reloads. Forms submit without the page flashing white. Data appears and updates as you interact.

That experience is closely tied to a concept called a **Single Page Application (SPA)**.

An SPA is not “one screen” and it’s not “a small site.” It’s a particular way of building a web application:

> The browser loads the application once, and from then on the app updates the UI dynamically without full page reloads.

To understand what that means (and why it matters), it helps to compare SPAs to traditional multi-page sites.

---

## 1) Traditional Websites (Multi-Page Applications)

In a traditional website:

1. You click a link to a new page
    
2. The browser sends a request to the server for a new HTML document
    
3. The server responds with a complete HTML page
    
4. The browser replaces the whole page and reloads everything (CSS, scripts as needed)
    
5. You see the new page
    

That flow is simple and robust. It’s still widely used and often the right choice for content-focused sites (blogs, documentation, news).

But it has some limitations for highly interactive applications:

- frequent full reloads can feel slow
    
- UI state resets easily (scroll position, open menus, draft text)
    
- the server must produce full page HTML for every navigation step
    

---

## 2) What Makes an SPA Different?

In an SPA, the server usually delivers:

- a small HTML shell (often just a `<div id="root"></div>`)
    
- a JavaScript bundle that contains the frontend app
    
- CSS and other assets
    

After that first load, most interaction happens like this:

1. User clicks something
    
2. React changes the UI (renders a different component view)
    
3. If data is needed, the app sends an HTTP request to an API endpoint
    
4. The page updates without a full reload
    

So instead of constantly requesting _new HTML pages_, an SPA mainly requests **data**, usually in JSON.

A useful slogan:

> Traditional sites navigate by loading new documents.  
> SPAs navigate by updating the UI.

---

## 3) “Single Page” Doesn’t Mean “One View”

The name can be confusing.

“Single Page Application” means:

- there is typically **one main HTML document** loaded initially
    

But the app can still have many “pages” from the user’s point of view:

- Home
    
- Dashboard
    
- Tasks
    
- Profile
    
- Settings
    

In an SPA, those “pages” are typically:

- different UI states
    
- different routes handled by the frontend router
    
- different component trees
    

So, you still design pages and navigation—just without fetching a new HTML file every time.

---

## 4) A Concrete Example: Navigating in an SPA

Imagine your Study Planner has two views:

- Today
    
- Add Task
    

In a traditional site:

- clicking “Add Task” might request `/add-task` from the server
    
- the server responds with a new HTML page
    

In an SPA:

- clicking “Add Task” updates the UI to show the Add Task component
    
- the browser URL may update (via a router), but the page doesn’t reload
    
- the app might fetch data in the background if needed
    

The user sees navigation, but the browser is not constantly replacing the document.

---

## 5) Why SPAs Feel Fast

SPAs often feel faster because:

- UI updates happen instantly in the browser
    
- only data is fetched (JSON), not entire documents
    
- once the JS bundle is loaded, navigating to new views doesn’t require reloading the app
    

However, SPAs also have tradeoffs (we’ll cover those too).

---

## 6) The Role of APIs in SPAs

A key feature of most SPAs is this separation:

- **Frontend**: responsible for UI and interactions
    
- **Backend**: provides data and performs secure actions (authentication, database writes)
    
- **Database**: stores persistent data
    

SPAs commonly communicate with the backend using API calls:

- `GET /tasks` → fetch list of tasks
    
- `POST /tasks` → create a task
    
- `PATCH /tasks/:id` → update a task
    
- `DELETE /tasks/:id` → remove a task
    

This separation is important because it mirrors how many real-world systems work and it matches your course stack (React + Python + PostgreSQL).

---

## 7) State and SPAs: Why They Belong Together

Because SPAs keep the application loaded, it becomes useful to store and manage UI state in the frontend:

- which view is currently active
    
- which user is logged in
    
- what tasks are loaded
    
- what’s in a form input
    
- whether a modal is open
    
- loading and error states
    

React excels here because it’s built around state-driven UI updates.

In a traditional site, a full page reload naturally “resets” state.  
In an SPA, you have to **manage state intentionally**—but you also get the benefit of a smoother experience.

---

## 8) Routing in SPAs (Preview)

Even without full reloads, SPAs can still have URLs like:

- `/tasks`
    
- `/tasks/123`
    
- `/profile`
    

This is usually handled by a client-side router (such as React Router). The router updates the displayed components based on the URL.

Important idea:

> The URL still matters. The difference is where routing is handled:  
> server-side routing (traditional) vs client-side routing (SPA).

In this course, you’ll use SPA routing patterns later once the core React basics are in place.

---

## 9) Tradeoffs: When SPAs Are Great (and When They Aren’t)

SPAs are excellent for:

- dashboards
    
- CRUD applications (task apps, admin panels)
    
- interactive tools
    
- apps that require login and personalized views
    

But they can be less ideal for:

- simple static sites
    
- content-first pages where SEO is crucial
    
- low-bandwidth contexts (large initial JS bundle)
    

In modern web development, there are hybrid approaches too (server-side rendering, static generation), but for learning full-stack fundamentals, SPAs are a great fit: they clearly show how the frontend, backend, and database collaborate.

---


---
# React: A Different Way to Think About UI (Expanded)

When you first learn web development, it’s natural to think of a webpage as a “thing” you build: you write HTML, the browser displays it, and then JavaScript reaches into the page and changes it when something happens.

That approach works. It’s how the web started, and it’s still useful.

But once you begin building interactive features—lists, forms, modals, filtering, sorting, user accounts—something starts to happen: the hardest part isn’t making the first version work. The hardest part is keeping the UI correct as the application grows.

React exists because large, interactive interfaces have a recurring problem:

> When data changes, the UI must change too—consistently, everywhere, without forgetting anything.

React offers a different mental model for solving that problem.

---

## 1) The “Manual DOM” Model vs the “State-Driven UI” Model

### In vanilla JavaScript, you often do this:

- Find an element
    
- Change its text
    
- Add or remove nodes
    
- Keep track of what is currently displayed
    
- Make sure all views stay consistent
    

Example mindset:

> “The user added a task. I must add a new `<li>` element.”

This works, but it’s easy to forget edge cases:

- What if the list is empty?
    
- What if the task count is shown in two places?
    
- What if a filter is active?
    
- What if the user is on a different view?
    

You can solve these problems in vanilla JS, but you end up building your own mini-framework: you invent patterns for rendering, state storage, updates, and DOM syncing.

---

### React flips the approach:

Instead of manually updating the DOM, you describe the UI you want for the current data:

> “Here is the list of tasks. The UI should reflect whatever is in this list.”

React then takes responsibility for keeping the DOM updated when the data changes.

This leads to the core React idea:

> **UI = f(state)**  
> The user interface is a function of application state.

---

## 2) What React Actually Does

React is a library for building user interfaces that helps you:

- break UI into components
    
- manage state
    
- render UI based on that state
    
- update the DOM efficiently when state changes
    

From a student perspective, React’s job is basically:

1. You describe the UI (using components + JSX/TSX)
    
2. You store data in state
    
3. When state changes, React updates the page to match
    

You don’t “tell React what to change.”  
You tell React what the UI should look like given the data.

---

## 3) Declarative vs Imperative (A Key Concept)

React is **declarative**. Vanilla DOM manipulation is often **imperative**.

### Imperative = “how to do it”

> “Create a list item. Set text. Append it. If empty, remove the empty message.”

### Declarative = “what the result should be”

> “Render this array of tasks as list items. If empty, show a message.”

That declarative approach is what makes complex UI easier to maintain.

---

## 4) A Small Comparison Example

### Vanilla JS style (simplified)

You might do something like:

- update array
    
- manually rebuild DOM
    

React style:

```tsx
function TaskList({ tasks }: { tasks: { id: string; title: string }[] }) {
  return (
    <ul>
      {tasks.map((t) => (
        <li key={t.id}>{t.title}</li>
      ))}
    </ul>
  );
}
```

If `tasks` changes, this output changes. You don’t directly manipulate `<li>` nodes.

---

## 5) React’s “Re-render” Is Not the Same as Reloading a Page

When React re-renders, it does **not** reload the browser page.

Instead:

- your component functions run again
    
- React compares what the UI should look like now vs before
    
- React updates only the necessary parts of the DOM
    

So “re-render” is more like:

> “Recalculate the UI and apply the minimal changes.”

This is what makes React feel fast and responsive.

---

## 6) React Encourages You to Model Your Data Properly

In React, you usually start by thinking:

- What data does this feature need?
    
- What shape is that data?
    
- Who owns it (which component)?
    
- Which parts of the UI depend on it?
    

For a task app, that might be:

```ts
type Task = {
  id: string;
  title: string;
  done: boolean;
  important: boolean;
};
```

Then your UI becomes a direct reflection of this data:

- done tasks render differently
    
- important tasks render bold
    
- task count updates automatically
    

React works best when you treat the data model as the “source of truth.”

---

## 7) The Component Tree: React as a System of Connected Pieces

React apps are built as a tree of components:

- `App`
    
    - `Header`
        
    - `TaskSummary`
        
    - `TaskList`
        
        - `TaskItem`
            
    - `AddTaskForm`
        

Each component has a responsibility:

- `TaskItem` displays one task
    
- `TaskList` maps tasks to items
    
- `App` might own the tasks state and define handlers
    

This tree structure makes it easier to reason about where logic lives.

---

## 8) The Real Problem React Solves: Consistency

The biggest challenge in UI development is not “adding a task.”  
It’s making sure every part of the UI stays consistent.

When tasks change:

- the list updates
    
- the count updates
    
- empty state disappears
    
- filter view updates
    
- sorting still works
    
- the “done” styling updates
    

In vanilla JS, you must remember to update each of these pieces.

In React:

- those pieces are derived from the same state
    
- when state changes, everything re-renders consistently
    

A React mindset is:

> If the UI is wrong, it means the state is wrong (or you’re rendering from the wrong state).

That’s a powerful debugging advantage.

---

## 9) React and SPAs: A Natural Fit

React is often used to build SPAs because:

- it handles complex UI changes smoothly
    
- it allows you to build “pages” as components
    
- it works well with API-driven data
    

In your course project:

- React renders UI
    
- Python backend provides JSON APIs
    
- PostgreSQL stores data
    
- React state updates based on API responses
    

React becomes the tool that ties UI logic to the app’s data flow.

---

## 10) What React Doesn’t Do (Important Clarity)

React does **not** automatically:

- fetch data (you still use `fetch`/axios)
    
- manage routing (you use a router library)
    
- validate backend data (you still need validation)
    
- replace the backend (you still build APIs)
    

React focuses on UI + state.

This is helpful because it keeps responsibilities clear.

---
# Components, JSX, and **TSX**: Building With Lego Bricks 

React encourages you to build UIs the way you’d build with Lego: small pieces that each do one job, assembled into larger structures. In React, those Lego bricks are **components**—and the most common way we describe what a component renders is with **JSX**.

When we add TypeScript into the mix, we start writing React components in a slightly different file format: **TSX**.

So in practice, this section is really about three related ideas:

- **Components**: reusable UI pieces
    
- **JSX**: HTML-like syntax for describing UI in JavaScript
    
- **TSX**: JSX + TypeScript together, so your UI code can be type-checked
    

---

## 1) Components: Your Own Reusable UI Tags

A React component is usually a function that returns UI:

```tsx
function Hello() {
  return <p>Hello!</p>;
}
```

You can use it like an HTML element:

```tsx
function App() {
  return (
    <div>
      <Hello />
      <Hello />
    </div>
  );
}
```

This is the “custom tag” moment: you created `<Hello />` as your own building block.

---

## 2) JSX: HTML-like Syntax Inside JavaScript

JSX lets you write UI in a way that resembles HTML:

```tsx
return (
  <section>
    <h2>Today</h2>
    <p>You have 3 tasks.</p>
  </section>
);
```

It’s important to remember: **JSX is not HTML**. Browsers can’t run JSX directly. Your build tools transform JSX into JavaScript that tells React what to create and update in the DOM.

A helpful mental model:

> JSX is a **syntax for describing UI**, not a separate language the browser understands.

---

## 3) So What Is TSX?

When you use React with TypeScript, you’ll write many components in files ending with:

- `.ts` for normal TypeScript code (no JSX inside)
    
- `.tsx` for TypeScript code **that contains JSX**
    

### Why do we need `.tsx`?

Because this line:

```tsx
return <h1>Hello</h1>;
```

contains something that _looks like_ HTML. Without TSX support, TypeScript would interpret `<h1>` as a comparison operator or invalid syntax. The `.tsx` extension tells the TypeScript compiler:

> “This file includes JSX syntax—parse it correctly.”

### Simple rule

- If a file contains JSX tags like `<div>...</div>`, make it **.tsx**
    
- If it contains only TypeScript logic (helpers, API code, types), make it **.ts**
    

---

## 4) What Changes When You Move From JSX to TSX?

JSX works in JavaScript projects.  
TSX works in TypeScript projects.

The UI code looks almost the same, but in TSX you get extra benefits:

### ✅ Type checking for component props

If you define a component’s props type, TypeScript can catch mistakes immediately.

```tsx
type TaskItemProps = {
  title: string;
  done: boolean;
};

function TaskItem({ title, done }: TaskItemProps) {
  return <li>{done ? "✅" : "⬜"} {title}</li>;
}
```

Now if someone tries:

```tsx
<TaskItem title="Read notes" done="nope" />
```

TypeScript warns you: `done` must be a boolean.

### ✅ Safer event handling

TSX lets you type event handlers (we’ll practice this soon):

```tsx
function handleSubmit(e: React.FormEvent) {
  e.preventDefault();
}
```

### ✅ Better editor help (autocomplete)

When you type `task.` your editor can show the exact fields available because TypeScript knows the object shape.

---

## 5) JSX/TSX Rules You’ll Use All the Time

These rules apply in both JSX and TSX.

### Rule 1: Return one root element

✅ Works:

```tsx
return (
  <div>
    <h1>Title</h1>
    <p>Text</p>
  </div>
);
```

If you don’t want a wrapper element, use a fragment:

```tsx
return (
  <>
    <h1>Title</h1>
    <p>Text</p>
  </>
);
```

### Rule 2: Use `className` (not `class`)

```tsx
<p className="card">Hello</p>
```

This becomes extra important later with Tailwind.

### Rule 3: Close all tags

```tsx
<img src="logo.png" alt="Logo" />
<input type="text" />
```

### Rule 4: Put JavaScript expressions inside `{ }`

```tsx
const name = "Ada";
return <p>Hello, {name}!</p>;
```

---

## 6) A Tiny TSX Example: The Study Planner Header

Here’s a realistic TSX component:

```tsx
type HeaderProps = {
  title: string;
};

function Header({ title }: HeaderProps) {
  return (
    <header>
      <h1>{title}</h1>
      <nav>
        <a href="#today">Today</a>
        <a href="#add">Add task</a>
      </nav>
    </header>
  );
}
```

And you can use it like:

```tsx
function App() {
  return (
    <main>
      <Header title="Study Planner" />
      {/* more components here */}
    </main>
  );
}
```

Notice:

- TSX lets us combine JSX UI and TypeScript types cleanly.
    
- `HeaderProps` makes the expected data explicit.
    

---

## 7) Lists in TSX: Data → UI

In your vanilla JS version, you created `<li>` elements manually. In TSX, you declare what the list should look like:

```tsx
type Task = { id: string; title: string; done: boolean };

type TaskListProps = {
  tasks: Task[];
};

function TaskList({ tasks }: TaskListProps) {
  return (
    <ul>
      {tasks.map((t) => (
        <li key={t.id}>
          {t.done ? "✅" : "⬜"} {t.title}
        </li>
      ))}
    </ul>
  );
}
```

TypeScript helps here too:

- `tasks` must be an array of `Task`
    
- `t.id`, `t.title`, `t.done` are guaranteed to exist
    

---

# Props: Passing Data Into Components 

Once you start building UIs out of components, you quickly run into a practical question:

> How does one component give information to another?

For example, a `TaskItem` component needs to know:

- the task title
    
- whether it’s done
    
- maybe whether it’s important
    
- maybe what happens when you click “Remove”
    

That information has to come from somewhere.

In React, the main way data moves from one component to another is through **props**.

---

## 1) What Are Props?

**Props** (short for “properties”) are the inputs to a component—like function parameters.

A component is a function. Props are what you pass into that function.

```tsx
type GreetingProps = {
  name: string;
};

function Greeting({ name }: GreetingProps) {
  return <p>Hello, {name}!</p>;
}
```

Using it:

```tsx
<Greeting name="Ada" />
<Greeting name="Linus" />
```

Each time you use the component, you can give it different inputs. That’s what makes components reusable.

### Mental model

- A component is like a **machine**
    
- Props are the **ingredients**
    
- The rendered UI is the **output**
    

---

## 2) Props Are Read-Only

A very important rule:

> A component must not change its own props.

Props are owned by the parent and passed down to the child. A child can _use_ props, but it should not modify them.

This protects your UI from becoming unpredictable.

If you need data to change, you use **state** (covered in the next section). Props are for _receiving_ data.

---

## 3) Parent → Child Data Flow (“One Way Data Flow”)

React has a key design principle:

> Data flows down the component tree.

That means:

- Parents pass props to children
    
- Children render based on those props
    
- Children can request changes by calling callbacks (more on this below)
    

Example:

```tsx
function App() {
  const course = "Fullstack Web Development";
  return <Header title={course} />;
}

type HeaderProps = { title: string };

function Header({ title }: HeaderProps) {
  return <h1>{title}</h1>;
}
```

`App` owns the data (`course`), and `Header` receives it through props.

This “one-way” rule is one of the reasons React apps stay manageable.

---

## 4) Typed Props in TSX: Your Safety Net

In TypeScript + React, we typically define prop types explicitly. This gives you fast feedback while coding.

```tsx
type TaskItemProps = {
  title: string;
  done: boolean;
};

function TaskItem({ title, done }: TaskItemProps) {
  return <li>{done ? "✅" : "⬜"} {title}</li>;
}
```

If you try to use it incorrectly:

```tsx
<TaskItem title={123} done="yes" />
```

TypeScript will warn you immediately.

That’s a big deal in group projects and larger codebases.

---

## 5) Props Can Be More Than Strings and Numbers

Props can be almost any kind of value:

- strings
    
- numbers
    
- booleans
    
- objects
    
- arrays
    
- functions (very important)
    
- even JSX elements (children)
    

### Example: Passing an object

```tsx
type Task = {
  id: string;
  title: string;
  done: boolean;
};

type TaskItemProps = {
  task: Task;
};

function TaskItem({ task }: TaskItemProps) {
  return <li>{task.done ? "✅" : "⬜"} {task.title}</li>;
}
```

Using it:

```tsx
<TaskItem task={{ id: "1", title: "Read notes", done: false }} />
```

This often scales better than passing many individual props.

---

## 6) Props vs State: A Clear Distinction

Students often mix these up. A practical way to separate them:

### Props

- come from the parent
    
- represent external input
    
- are read-only
    
- make a component reusable
    

### State

- lives inside a component (or is managed higher up)
    
- changes over time
    
- causes re-renders when updated
    
- makes the app interactive
    

A component might use both:

- props to receive a task
    
- state to track something like “is editing mode open?”
    

---

## 7) Passing Functions as Props (How Children Trigger Changes)

So far, props look like data. But one of the most powerful patterns is:

> Pass a function as a prop.

This is how a child component can “tell” the parent something happened.

### Why do we need this?

Remember: children shouldn’t modify props. But users interact with child components (click “Remove”, toggle “Done”). Those interactions need to update the parent’s state.

Solution: the parent passes a callback.

#### Example: TaskItem with a remove button

```tsx
type Task = { id: string; title: string; done: boolean };

type TaskItemProps = {
  task: Task;
  onRemove: (taskId: string) => void;
};

function TaskItem({ task, onRemove }: TaskItemProps) {
  return (
    <li>
      {task.title}
      <button type="button" onClick={() => onRemove(task.id)}>
        Remove
      </button>
    </li>
  );
}
```

Parent:

```tsx
function App() {
  const [tasks, setTasks] = useState<Task[]>([
    { id: "1", title: "Read notes", done: false },
  ]);

  function handleRemove(taskId: string) {
    setTasks(tasks.filter((t) => t.id !== taskId));
  }

  return (
    <ul>
      {tasks.map((t) => (
        <TaskItem key={t.id} task={t} onRemove={handleRemove} />
      ))}
    </ul>
  );
}
```

### What’s happening conceptually

- Parent owns the data and state (`tasks`)
    
- Child displays data (`TaskItem`)
    
- Child reports events upward (`onRemove`)
    
- Parent updates state, which updates UI
    

This is a central React pattern.

---

## 8) `children`: Passing UI Into Components

React components can also receive a special prop called `children`, which lets you wrap content:

```tsx
type CardProps = {
  children: React.ReactNode;
};

function Card({ children }: CardProps) {
  return <div className="card">{children}</div>;
}
```

Using it:

```tsx
<Card>
  <h3>Today</h3>
  <p>3 tasks remaining</p>
</Card>
```

This is how you build reusable layout components.

---

## 9) Common Beginner Mistakes With Props

### Mistake 1: Trying to change props

```tsx
function TaskItem({ task }: TaskItemProps) {
  task.done = true; // ❌ Don’t do this
  return <li>{task.title}</li>;
}
```

Instead:

- store mutable data in state in the parent
    
- update state using setters
    
- pass updated data back down as props
    

### Mistake 2: Passing too many tiny props

This can get messy:

```tsx
<TaskItem title={t.title} done={t.done} important={t.important} due={t.due} />
```

Sometimes it’s better to pass the whole object:

```tsx
<TaskItem task={t} />
```

### Mistake 3: Forgetting function props are _called_

```tsx
<button onClick={onRemove(task.id)}>Remove</button> // ❌ calls immediately
```

Correct:

```tsx
<button onClick={() => onRemove(task.id)}>Remove</button> // ✅ called on click
```

---

## State: Data That Changes Over Time 

In the earlier JavaScript build-along, you had a key problem to solve:

- You had some data (an array of tasks).
    
- The user changed that data (adding/removing/toggling tasks).
    
- The page needed to update to match the new data.
    

In vanilla JavaScript, you solved this by manually updating the DOM:

- create elements
    
- remove elements
    
- change text
    
- re-render sections
    

React takes a different approach. It asks you to treat your UI like a “live view” of your data:

> If the data changes, the UI should automatically update.

That is what **state** is for.

---

## 1) What Is State?

**State** is data that belongs to a component and can change over time.

Examples of state:

- the current value of a counter
    
- the contents of a form field
    
- a list of tasks loaded from an API
    
- whether a modal is open or closed
    
- loading/error status of a network request
    

A useful definition:

> State is the memory of your UI.

If the user interacts with the page and something should be remembered, that’s often state.

---

## 2) `useState`: The Most Common React Hook

In React, you create state using the `useState` hook.

```tsx
import { useState } from "react";

function Counter() {
  const [count, setCount] = useState(0);

  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={() => setCount(count + 1)}>+</button>
    </div>
  );
}
```

### How to read this line

```ts
const [count, setCount] = useState(0);
```

- `count` is the current state value
    
- `setCount` is the function to update it
    
- `0` is the initial value
    

React will re-render the component whenever you call `setCount`.

---

## 3) The React Loop: Render → Event → State Update → Re-render

React apps typically follow a loop:

1. React renders your component using current state
    
2. The user does something (click, type, submit)
    
3. Your event handler updates state
    
4. React re-renders the component with the new state
    

This loop is the heartbeat of a React application.

Unlike vanilla DOM code, you don’t “manually redraw” the UI. You update state, and React redraws the UI for you.

---

## 4) State Lives in a Specific Place (Who Owns It?)

A big design question in React is:

> Which component should own which state?

Rule of thumb:

- State should live in the **lowest common ancestor** that needs it.
    

Example:

- If only `AddTaskForm` needs the text input value, keep it in `AddTaskForm`.
    
- If both `TaskList` and `TaskCounter` need access to tasks, the tasks state should live in a parent component above both.
    

This avoids duplication and confusion.

---

## 5) Different Kinds of State You’ll Use

### A) Primitive state (number/string/boolean)

```tsx
const [isOpen, setIsOpen] = useState(false);
const [name, setName] = useState("");
const [count, setCount] = useState(0);
```

### B) Object state (careful!)

```tsx
const [profile, setProfile] = useState({ name: "Ada", age: 20 });
```

### C) Array state (very common)

```tsx
type Task = { id: string; title: string; done: boolean };
const [tasks, setTasks] = useState<Task[]>([]);
```

Arrays and objects are common in apps, but you must update them in an immutable way (explained next).

---

## 6) Updating State: Treat It as Immutable

A common beginner mistake is trying to mutate state directly.

### ❌ Don’t do this

```tsx
tasks.push(newTask);     // mutates array
setTasks(tasks);         // may not trigger correct updates
```

React expects state updates to create a **new** array/object reference.

### ✅ Do this instead (create a new array)

```tsx
setTasks([...tasks, newTask]);
```

### ✅ Updating one item in an array (map)

```tsx
setTasks(tasks.map(t => t.id === id ? { ...t, done: !t.done } : t));
```

### ✅ Removing an item (filter)

```tsx
setTasks(tasks.filter(t => t.id !== id));
```

This is the same idea you used in vanilla JS when you re-assigned `tasks = tasks.filter(...)`. In React, immutability helps React detect changes reliably.

---

## 7) State Updates May Be Asynchronous (Use the “Updater Form”)

Sometimes you update state based on the previous value. In those cases, use the updater function pattern:

```tsx
setCount((prev) => prev + 1);
```

Why?

- If multiple updates happen quickly, using `count + 1` can be based on an older value.
    
- The updater form guarantees you use the latest state.
    

This matters especially with rapid clicks or multiple state updates.

---

## 8) Controlled Inputs: State + Forms

One of the most important uses of state is form input.

A **controlled input** means:

- the input value comes from React state
    
- changes update state via `onChange`
    

```tsx
function NameField() {
  const [name, setName] = useState("");

  return (
    <div>
      <label>
        Name
        <input
          value={name}
          onChange={(e) => setName(e.target.value)}
          placeholder="Type your name"
        />
      </label>

      <p>Hello, {name}</p>
    </div>
  );
}
```

This pattern is extremely common because it keeps the UI and data in sync.

---

## 9) A Realistic Example: Tasks State in the Study Planner

Here’s how your task array becomes state:

```tsx
type Task = {
  id: string;
  title: string;
  done: boolean;
  important: boolean;
};

function App() {
  const [tasks, setTasks] = useState<Task[]>([
    { id: "1", title: "Read notes", done: false, important: false },
  ]);

  function addTask(title: string) {
    const newTask: Task = {
      id: crypto.randomUUID(),
      title,
      done: false,
      important: false,
    };

    setTasks([...tasks, newTask]);
  }

  return (
    <main>
      <h1>Study Planner</h1>
      <p>You have {tasks.length} tasks</p>

      <ul>
        {tasks.map((t) => (
          <li key={t.id}>
            {t.done ? "✅" : "⬜"} {t.title}
          </li>
        ))}
      </ul>

      <button onClick={() => addTask("New task")}>Add example task</button>
    </main>
  );
}
```

Notice:

- `tasks` is the single source of truth
    
- UI is derived from `tasks`
    
- Adding a task updates state, which re-renders the list
    

---

## 10) State vs Props (A quick connection)

It helps to connect the ideas:

- **Props** are inputs given to a component
    
- **State** is data owned by a component
    

Often:

- the parent stores state
    
- the parent passes parts of that state as props to children
    
- children call callbacks to request updates
    

This pattern is what you’ll use for most of the course project.

---

## 11) Common Beginner Mistakes With State

### Mistake 1: Mutating state

```tsx
tasks.push(task); // ❌
```

Use `setTasks([...tasks, task])`.

### Mistake 2: Setting state in render

```tsx
if (tasks.length === 0) setTasks([...]); // ❌ infinite loop risk
```

State updates belong in event handlers or effects (you’ll learn effects later).

### Mistake 3: Too much state

Not everything needs to be state. If something can be computed from state/props, compute it:

```tsx
const doneCount = tasks.filter(t => t.done).length;
```

No need to store `doneCount` separately.

---

# Events in React: Responding to the User

A web application becomes an application when users can interact with it.

They click buttons, type into inputs, submit forms, open menus, and toggle checkboxes. Every one of those actions produces an **event**. In React, events are the bridge between what the user does and how your app’s **state** changes.

A useful way to think about it:

> **Events don’t change the UI directly.**  
> Events trigger code, and that code usually updates **state**.  
> Updating state causes React to re-render, which updates the UI.

So the flow becomes:

**User action → event handler → state update → UI update**

---

## 1) React Events vs “Plain DOM Events”

If you worked with vanilla JS, you might have written:

```js
button.addEventListener("click", () => { ... });
```

In React, you attach handlers directly in JSX:

```tsx
<button onClick={() => { /* handle click */ }}>
  Click me
</button>
```

Key differences you’ll notice:

- You don’t “find the element” with `querySelector`.
    
- You attach the handler where the element is defined.
    
- Your handler usually updates state rather than manually updating DOM nodes.
    

React uses a system called **Synthetic Events** (an abstraction over browser events). For most student-level work, you can treat them like normal events—just know they behave consistently across browsers.

---

## 2) Event Handler Basics: Functions, Not Function Calls

In React, you pass a **function** as the event handler.

✅ Correct:

```tsx
<button onClick={handleClick}>Add</button>
```

❌ Incorrect (this runs immediately during render):

```tsx
<button onClick={handleClick()}>Add</button>
```

If you need to pass arguments, wrap the call in an arrow function:

```tsx
<button onClick={() => handleRemove(task.id)}>
  Remove
</button>
```

This ensures the function runs **only when the event occurs**, not when the component renders.

---

## 3) The Most Common Events You’ll Use

### Click

```tsx
<button onClick={() => setCount((c) => c + 1)}>
  +
</button>
```

### Change (inputs)

In React, for text inputs you usually use `onChange` to react to user typing:

```tsx
<input
  value={title}
  onChange={(e) => setTitle(e.target.value)}
/>
```

### Submit (forms)

```tsx
<form onSubmit={handleSubmit}>
  ...
</form>
```

### Keyboard events

```tsx
<input
  onKeyDown={(e) => {
    if (e.key === "Enter") {
      // do something
    }
  }}
/>
```

You won’t need keyboard events constantly, but they’re useful for accessibility and good UX.

---

## 4) Typing Events in TypeScript (TSX)

Because we’re using TypeScript, we want handlers to be type-safe. The most common event types you’ll see are:

### Form submit

```tsx
function handleSubmit(e: React.FormEvent<HTMLFormElement>) {
  e.preventDefault();
}
```

Often you’ll see a simpler form too:

```tsx
function handleSubmit(e: React.FormEvent) {
  e.preventDefault();
}
```

### Input change

```tsx
function handleTitleChange(e: React.ChangeEvent<HTMLInputElement>) {
  setTitle(e.target.value);
}
```

### Button click

Most click handlers don’t need the event object at all:

```tsx
function handleAdd() {
  setCount((c) => c + 1);
}
```

If you _do_ need it:

```tsx
function handleClick(e: React.MouseEvent<HTMLButtonElement>) {
  // e.currentTarget refers to the button
}
```

**Tip for beginners:** If the event typing feels heavy at first, it’s OK to start with minimal typing and add types as you get comfortable. But for forms and inputs, typing is helpful because it prevents common mistakes.

---

## 5) `event.target` vs `event.currentTarget`

This can be confusing early on.

- `event.target` = the element that _triggered_ the event (could be a child element)
    
- `event.currentTarget` = the element your handler is attached to
    

In many cases, you’ll use `e.target.value` for inputs, but TypeScript sometimes complains because `target` can be generic. A safe pattern is to type the event properly, or use `currentTarget` where appropriate.

Example with typed input event:

```tsx
function handleChange(e: React.ChangeEvent<HTMLInputElement>) {
  setTitle(e.target.value); // OK because the event is typed
}
```

---

## 6) React Pattern: Handlers Often Live Near the State

A common beginner confusion is where to write event handlers.

A good rule:

> Put event handlers in the component that owns the state being updated.

Example: If `App` owns the `tasks` state, then `addTask`, `removeTask`, `toggleTask` usually live in `App`, and are passed down as props.

This keeps your data flow clean:

- state is owned in one place
    
- children call callbacks to request updates
    

---

## 7) A Realistic Example: Add Task Form (Controlled Input + Submit)

Here’s the “core loop” in React:

```tsx
type Task = { id: string; title: string; done: boolean };

function App() {
  const [tasks, setTasks] = React.useState<Task[]>([]);
  const [title, setTitle] = React.useState("");

  function handleSubmit(e: React.FormEvent) {
    e.preventDefault();

    const trimmed = title.trim();
    if (!trimmed) return;

    const newTask: Task = {
      id: crypto.randomUUID(),
      title: trimmed,
      done: false,
    };

    setTasks((prev) => [...prev, newTask]);
    setTitle("");
  }

  return (
    <main>
      <h1>Study Planner</h1>

      <form onSubmit={handleSubmit}>
        <label>
          Task title
          <input
            value={title}
            onChange={(e) => setTitle(e.target.value)}
            placeholder="e.g., Finish lab"
          />
        </label>

        <button type="submit">Add</button>
      </form>

      <ul>
        {tasks.map((t) => (
          <li key={t.id}>{t.title}</li>
        ))}
      </ul>
    </main>
  );
}
```

What to notice:

- The input is controlled by `title` state.
    
- Typing triggers `onChange`, which updates `title`.
    
- Submitting triggers `onSubmit`, which creates a new task and updates `tasks`.
    
- Updating `tasks` causes a re-render and the list updates automatically.
    

This is the React way: no manual DOM manipulation.

---

## 8) Another Realistic Example: Toggle Done (Events + Immutable Updates)

```tsx
function toggleTask(taskId: string) {
  setTasks((prev) =>
    prev.map((t) =>
      t.id === taskId ? { ...t, done: !t.done } : t
    )
  );
}

<ul>
  {tasks.map((t) => (
    <li key={t.id}>
      <label>
        <input
          type="checkbox"
          checked={t.done}
          onChange={() => toggleTask(t.id)}
        />
        {t.title}
      </label>
    </li>
  ))}
</ul>
```

What this shows:

- The checkbox is controlled by state (`checked={t.done}`)
    
- `onChange` updates the state
    
- React updates the checkbox and UI automatically
    

---

## 9) Preventing Default Browser Behavior

In plain HTML:

- form submit reloads the page
    
- clicking a link navigates away
    

In React SPAs, you often prevent these default behaviors and handle navigation or submission yourself.

For forms:

```tsx
function handleSubmit(e: React.FormEvent) {
  e.preventDefault();
  // handle data in JS
}
```

For link-like buttons (often better to use `<button>`):

```tsx
<button type="button" onClick={...}>Open modal</button>
```

---

## 10) Common Beginner Mistakes With React Events

### Mistake 1: Calling the handler instead of passing it

```tsx
<button onClick={removeTask(task.id)}>Remove</button> // ❌
```

Correct:

```tsx
<button onClick={() => removeTask(task.id)}>Remove</button> // ✅
```

### Mistake 2: Using uncontrolled inputs accidentally

If you want React to control an input, you must provide `value` and `onChange`. Otherwise, you may end up with confusing behavior.

### Mistake 3: Mutating state inside event handlers

```tsx
tasks.push(newTask); // ❌
setTasks(tasks);
```

Correct:

```tsx
setTasks((prev) => [...prev, newTask]);
```

### Mistake 4: Forgetting `type="button"`

In a form, a plain `<button>` defaults to submit.

If you have extra buttons inside a form:

```tsx
<button type="button" onClick={resetForm}>Clear</button>
```

---

## 11) Connecting Events to the Component Tree (The “Callback Prop” Pattern)

Often events happen deep in the UI, but state is owned higher up.

That’s where callback props come in:

- Parent owns state and update functions
    
- Child calls the function when an event happens
    

Example:

```tsx
type TaskItemProps = {
  title: string;
  onRemove: () => void;
};

function TaskItem({ title, onRemove }: TaskItemProps) {
  return (
    <li>
      {title}
      <button type="button" onClick={onRemove}>Remove</button>
    </li>
  );
}
```

Parent:

```tsx
<TaskItem title={t.title} onRemove={() => removeTask(t.id)} />
```

This pattern is the backbone of React apps.


---

# Putting It All Together: Your Study Planner, Re-imagined in React

Think back to your vanilla JavaScript version:

- you had an array of tasks
    
- you rendered tasks by creating DOM nodes
    
- when tasks changed, you re-rendered manually
    

In React, the same idea becomes simpler:

- tasks are state: `const [tasks, setTasks] = useState<Task[]>([])`
    
- UI maps tasks to components: `tasks.map(...)`
    
- adding/removing tasks updates state, not the DOM directly
    

You’re still solving the same problem.  
You’re just using a tool that makes the solution more scalable.
