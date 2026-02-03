# daisyUI: Ready-Made Components on Top of Tailwind (Introduction + Examples)

[Daisy UI]([Tailwind CSS Component Library ⸺ daisyUI](https://daisyui.com/?lang=en))

When you first use Tailwind, you gain a powerful styling toolkit—but you may also notice something:

Even with utility classes, building common UI elements still takes time.

- Buttons need hover and focus styles
    
- Cards need spacing and borders
    
- Inputs need consistent sizing and focus rings
    
- Alerts, modals, dropdowns, navbars… all have repeated patterns
    

This is where **daisyUI** fits in.

> **daisyUI is a component library built on top of Tailwind CSS.**  
> It gives you pre-styled UI components using simple class names like `btn`, `card`, `input`, and `alert`.

You can think of it like this:

- **Tailwind** = LEGO bricks (small utilities)
    
- **daisyUI** = pre-built LEGO sets (buttons, cards, navbars, modals)
    

Both are still Tailwind-compatible. daisyUI doesn’t replace Tailwind—it builds on it.

---

## 1) What daisyUI Is (and What It Isn’t)

### daisyUI is…

- A Tailwind plugin that provides **component classes**
    
- A theme system (light/dark + many preset themes)
    
- A way to build consistent UI quickly without writing lots of custom CSS
    

### daisyUI is not…

- A separate CSS framework with its own rules
    
- A JavaScript component library (most components are CSS + HTML structure)
    
- A replacement for understanding HTML/CSS basics
    

daisyUI mostly works by giving you **meaningful class names** that bundle many Tailwind utilities into one.

---

## 2) Why We Use daisyUI in This Course

In a beginner-friendly course, time is precious. We want students to spend energy on:

- React component structure
    
- TypeScript correctness
    
- state and events
    
- API integration
    
- data flow and debugging
    

…not on reinventing button styles for the fifth time.

daisyUI helps you ship something that looks clean and consistent early, which improves motivation and makes lab projects feel “real”.

---

## 3) The daisyUI Mindset: Semantic UI Classes

With Tailwind alone, a button might look like this:

```html
<button class="px-4 py-2 rounded-lg bg-blue-600 text-white hover:bg-blue-700">
  Add
</button>
```

With daisyUI:

```html
<button class="btn btn-primary">Add</button>
```

### What changed?

- You’re describing intent: _this is a primary button_
    
- Consistent styling is handled for you
    
- You still can add Tailwind utilities when needed
    

---

## 4) Using daisyUI in React (TSX)

In TSX, remember: use `className`:

```tsx
export function AddButton() {
  return (
    <button className="btn btn-primary">
      Add task
    </button>
  );
}
```

daisyUI works well with React because you’re already thinking in components.

---

## 5) Themes: A Quick Way to Change the Look

daisyUI includes built-in themes (light/dark and many others). You can switch theme by setting a `data-theme` attribute.

Example (in `index.html` or a root element):

```html
<html data-theme="light">
```

Switch to dark:

```html
<html data-theme="dark">
```

Or you can set it on a wrapper:

```html
<div data-theme="light">
  ...
</div>
```

### Why this is nice

- You get a coherent color system without manual design work
    
- You can offer theme switching later with a single attribute change
    

---

## 6) Core Components You’ll Use a Lot (With Examples)

Below are common daisyUI building blocks that map nicely to course project needs.

---

### A) Buttons

```html
<button class="btn">Default</button>
<button class="btn btn-primary">Primary</button>
<button class="btn btn-secondary">Secondary</button>
<button class="btn btn-accent">Accent</button>
<button class="btn btn-outline">Outline</button>
<button class="btn btn-ghost">Ghost</button>
<button class="btn btn-sm">Small</button>
<button class="btn btn-disabled" disabled>Disabled</button>
```

#### Why it matters

Buttons become consistent across your app without manual styling.

---

### B) Cards (Perfect for “sections”)

```html
<div class="card bg-base-100 shadow">
  <div class="card-body">
    <h2 class="card-title">Today</h2>
    <p>You have 3 tasks.</p>
  </div>
</div>
```

#### What to notice

- `card` sets the basic container style
    
- `card-body` gives spacing and layout
    
- `bg-base-100` uses the theme’s base background color
    

---

### C) Inputs and Forms

```html
<label class="form-control w-full max-w-xs">
  <div class="label">
    <span class="label-text">Task title</span>
  </div>
  <input type="text" placeholder="e.g., Finish exercises" class="input input-bordered w-full" />
</label>
```

#### Why it’s useful

- Inputs look good immediately
    
- Labels and spacing are handled cleanly
    

---

### D) Alerts (Errors / Success messages)

```html
<div class="alert alert-success">
  <span>Task created successfully!</span>
</div>

<div class="alert alert-error">
  <span>Title is required.</span>
</div>
```

In a course project, alerts are great for:

- validation feedback
    
- API errors
    
- success confirmations
    

---

### E) Badges (Counts and labels)

```html
<h2 class="text-xl font-semibold">
  Today <span class="badge badge-neutral">3</span>
</h2>

<span class="badge badge-warning">Important</span>
```

Badges are perfect for task counts and status markers.

---

### F) Navbar (App layout)

```html
<div class="navbar bg-base-100 shadow">
  <div class="flex-1">
    <a class="btn btn-ghost text-xl">Study Planner</a>
  </div>
  <div class="flex-none gap-2">
    <button class="btn btn-ghost">Today</button>
    <button class="btn btn-ghost">Add task</button>
  </div>
</div>
```

---

### G) Modal (Useful later in the course)

daisyUI provides a modal pattern using HTML structure + classes. For example:

```html
<button class="btn" onclick="my_modal.showModal()">Open modal</button>

<dialog id="my_modal" class="modal">
  <div class="modal-box">
    <h3 class="font-bold text-lg">Hello!</h3>
    <p class="py-4">This is a modal box.</p>
    <div class="modal-action">
      <form method="dialog">
        <button class="btn">Close</button>
      </form>
    </div>
  </div>
</dialog>
```

#### Explanation

- Uses the native HTML `<dialog>` element
    
- Styling comes from daisyUI classes
    
- This becomes very handy for “edit task”, “confirm delete”, etc.
    

(When using React, you typically control modal open/close with state, but the structure remains similar.)

---

## 7) Combining daisyUI and Tailwind

This is the best part: you can use daisyUI for the base component styling and Tailwind utilities for layout tweaks.

Example:

```html
<button class="btn btn-primary w-full md:w-auto">
  Add task
</button>
```

- `btn btn-primary` = daisyUI component style
    
- `w-full md:w-auto` = Tailwind responsiveness
    

A good rule of thumb:

- Use daisyUI to get consistent components quickly
    
- Use Tailwind utilities for spacing, layout, responsiveness, and minor adjustments
    

---

## 8) Common Beginner Pitfalls

### “Why don’t the classes work?”

Usually one of these:

- Tailwind isn’t configured correctly
    
- daisyUI plugin isn’t enabled
    
- you forgot to rebuild/restart dev server after config changes
    

### “Which should I use: Tailwind or daisyUI?”

Use both:

- daisyUI for component styling
    
- Tailwind for layout control and custom tweaks
    

### “Can I customize themes?”

Yes—later you can choose a theme and optionally customize colors, but early on it’s best to use defaults so you can focus on programming concepts.

---

