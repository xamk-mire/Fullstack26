# Vite: A Fast, Friendly Development Environment

[Vite](https://vite.dev/)

When you built pages in Week 1 & 2, you could open `index.html` directly in a browser and everything worked. That’s great for learning fundamentals—but as soon as you start building modern applications with React, TypeScript, and multiple files, you need a better environment:

* a way to bundle code into something the browser can run
* a development server that automatically reloads when you save changes
* support for TypeScript and TSX
* a way to import files (components, CSS) cleanly
* helpful error messages when something breaks

That’s what **Vite** provides.

Vite (pronounced “veet”) is a modern tool for developing frontend applications. It gives you a fast dev server and a build process that prepares your app for production.

---

## 1) What Vite Does (In Plain Terms)

Vite helps you with two main phases of frontend development:

### Development (when you’re coding)

* Runs a **dev server** on your computer
* Updates the browser instantly when you save (Hot Module Replacement)
* Understands modern imports (`import ... from ...`)
* Supports TypeScript + TSX out of the box (with templates)
* Shows clear error messages in the browser and terminal

### Production (when you want to publish)

* Bundles and optimizes your code into a `dist/` folder
* Minifies assets and prepares your app to be deployed

A helpful mental model:

> Vite is like a “workbench” for building your app while you code, and a “packaging machine” when you’re ready to ship.

---

## 2) Why We Use Vite in This Course

React + TypeScript projects involve many files:

* components in `.tsx`
* helper functions in `.ts`
* CSS files
* images and other assets

The browser cannot directly run:

* TypeScript
* TSX (JSX inside TypeScript)
* module imports in the way we write them in modern projects

Vite sets everything up so you can write modern code and still run it in the browser easily.

---

## 3) Creating a Vite Project (React + TypeScript)

In a terminal, run:

```bash
npm create vite@latest
```

It will ask questions such as:

* Project name
* Framework (choose React)
* Variant (choose TypeScript)

Then:

```bash
cd your-project-name
npm install
npm run dev
```

### What these commands mean

* `npm create vite@latest`: scaffolds a new project with sane defaults
* `npm install`: downloads dependencies (React, Vite, etc.)
* `npm run dev`: starts the development server

You’ll see output similar to:

* a local URL (often `http://localhost:5173/`)

Open that URL in your browser to see your app.

---

## 4) The Most Important Vite Superpower: Instant Feedback

When the dev server is running:

* You edit a file
* Save it
* The browser updates immediately

This is much faster than manually refreshing, and it’s essential for React development.

This is called **Hot Module Replacement (HMR)**:

* Vite replaces only the modules that changed
* It tries to keep the app running without a full reload

---

## 5) Understanding the Vite Project Structure

A typical Vite + React + TS project looks like this:

```
my-app/
  index.html
  package.json
  vite.config.ts
  src/
    main.tsx
    App.tsx
    index.css
  public/
```

### What each part is

**`index.html`**

* The single HTML file (SPA “shell”)
* Contains the root element where React attaches

**`src/main.tsx`**

* The entry point: this is where React is started and mounted

**`src/App.tsx`**

* Your main UI component (you will add more components over time)

**`src/index.css`**

* Global styles (later: Tailwind base)

**`public/`**

* Static files that should be copied as-is (favicons, static images)

---

## 6) Example: “Hello React” in Vite

Open `src/App.tsx` and replace its contents with:

```tsx
function App() {
  return (
    <main>
      <h1>Hello from Vite + React + TypeScript</h1>
      <p>If you can see this, your dev environment works.</p>
    </main>
  );
}

export default App;
```

### Explanation

* `App` is a React component
* It returns JSX (TSX file)
* `export default App` allows other files to import it

Now save and check the browser—your changes should appear instantly.

---

## 7) Example: Creating Your First Component File

Create a new file:

`src/Greeting.tsx`

```tsx
type GreetingProps = {
  name: string;
};

export function Greeting({ name }: GreetingProps) {
  return <p>Hello, {name}!</p>;
}
```

Now use it in `App.tsx`:

```tsx
import { Greeting } from "./Greeting";

function App() {
  return (
    <main>
      <h1>Study Planner</h1>
      <Greeting name="Ada" />
      <Greeting name="Linus" />
    </main>
  );
}

export default App;
```

### What this demonstrates

* **ES module imports** (`import { ... } from ...`)
* component reuse
* TypeScript props

Vite makes this workflow feel natural, because it understands module imports and serves updated code automatically.

---

## 8) Example: Importing CSS in Vite

In Vite, CSS can be imported into your code (commonly in `main.tsx` or a component).

Open `src/index.css` and add:

```css
main {
  max-width: 900px;
  margin: 0 auto;
  padding: 16px;
  font-family: system-ui, -apple-system, Segoe UI, Roboto, Arial, sans-serif;
}
```

Make sure `src/main.tsx` includes:

```ts
import "./index.css";
```

(Usually it’s already there.)

### Explanation

* Vite bundles CSS into your app during development and build
* You can keep styles modular and organized

---

## 9) Example: Environment Variables (Preview)

Later in the course, your frontend needs to know where the backend API is.

Vite supports environment variables with a special rule:

> Variables that should be available in the frontend must start with `VITE_`.

Create a file:
`.env`

```env
VITE_API_BASE_URL=http://localhost:8000
```

Use it:

```ts
const baseUrl = import.meta.env.VITE_API_BASE_URL;
```

### Explanation

* `import.meta.env` is how Vite exposes environment variables
* the `VITE_` prefix protects you from accidentally exposing secrets

(Important: never put real secrets in frontend env vars.)

---

## 10) “Dev vs Build”: Two Different Worlds

### `npm run dev`

* Runs your app locally for development
* Fast, flexible, not optimized for production

### `npm run build`

* Produces an optimized production build in `dist/`
* This folder can be deployed to a static hosting service

### `npm run preview`

* Lets you preview the production build locally

These commands are part of the workflow you’ll reuse in every frontend sprint.

---

## 11) Common Beginner Issues (and How to Think About Them)

### “My changes don’t appear”

* Is `npm run dev` running?
* Did you save the file?
* Did you edit the correct file under `src/`?

### “Cannot find module…”

* Check the import path (`./Greeting` vs `../Greeting`)
* Verify the filename and capitalization match

### “White page, red errors”

* This is normal while learning—read the error message carefully
* Vite’s overlay tells you which file and line caused the crash
