# Tailwind CSS: Styling With Utility Classes 

[Tailwind CSS](https://tailwindcss.com/)

So far, you’ve styled pages by writing CSS rules like:

```css
section {
  border: 1px solid #ddd;
  padding: 16px;
  border-radius: 12px;
}
```

That approach is totally valid—and you should understand it. But as projects grow, you often run into a new kind of complexity:

- You spend time inventing class names (`.card`, `.card2`, `.card-primary`, `.card-secondary`…)
    
- Your CSS file grows and becomes harder to navigate
    
- You tweak styles in one place and accidentally affect something elsewhere
    
- You jump back and forth between HTML and CSS constantly
    

**Tailwind CSS** offers a different approach.

> Instead of writing lots of custom CSS rules, you style elements using small, composable “utility classes” directly in your HTML/TSX.

Tailwind doesn’t remove CSS—it changes _how you apply it_.

---

## 1) What Tailwind Is (and What It Isn’t)

### Tailwind is…

- A **utility-first CSS framework**
    
- A large set of predefined classes for:
    
    - spacing (`p-4`, `mt-2`)
        
    - layout (`flex`, `grid`, `justify-between`)
        
    - typography (`text-lg`, `font-semibold`)
        
    - color (`bg-blue-500`, `text-gray-700`)
        
    - borders and shadows (`rounded-xl`, `shadow`)
        
    - responsive design (`md:flex`, `lg:p-8`)
        
- A tool that encourages building UI by composing utilities
    

### Tailwind is not…

- A component library (that’s where daisyUI helps)
    
- “No CSS ever” (you can still write CSS when needed)
    
- Inline styles (it’s still CSS classes, just very small and focused)
    

---

## 2) Why Utility Classes Work Well in React Projects

React components are already about bundling UI into small pieces. Tailwind matches that mindset:

- Each component contains its **structure and styling** together
    
- You can move a component without worrying about missing CSS files
    
- It reduces “mystery styles” (where did this come from?)
    

A practical mental model:

> Tailwind makes styling feel like building with Lego:  
> you assemble small pieces into a larger design.

---

## 3) The Tailwind “Vocabulary” (Key Ideas)

Tailwind classes usually follow patterns:

### Spacing

- `p-4` = padding on all sides
    
- `px-4` = padding left and right
    
- `py-2` = padding top and bottom
    
- `mt-6` = margin-top
    
- `gap-3` = gap between items in flex/grid
    

### Layout

- `flex`, `grid`
    
- `items-center`, `justify-between`
    
- `w-full`, `max-w-3xl`
    

### Typography

- `text-sm`, `text-lg`, `text-2xl`
    
- `font-medium`, `font-bold`
    
- `leading-relaxed`
    

### Borders & Shadows

- `border`, `border-gray-200`
    
- `rounded-lg`, `rounded-2xl`
    
- `shadow`, `shadow-md`
    

### Colors (examples)

- `bg-white`, `bg-gray-50`
    
- `text-gray-900`, `text-blue-600`
    

You don’t need to memorize everything. You learn Tailwind like a language: a little at a time, as you build.

---

## 4) Example: From Plain HTML to Tailwind

### Without Tailwind

```html
<section>
  <h2>Today</h2>
  <p>You have 3 tasks.</p>
</section>
```

### With Tailwind

```html
<section class="border border-gray-200 rounded-xl p-4 bg-gray-50">
  <h2 class="text-xl font-semibold mb-2">Today</h2>
  <p class="text-gray-700">You have 3 tasks.</p>
</section>
```

### What changed?

- We added border, padding, rounding, and background via classes
    
- We styled the heading and text without writing a separate CSS file
    
- Each class does one small job
    

---

## 5) Tailwind in React + TypeScript (TSX)

Tailwind is commonly used inside TSX like this:

```tsx
function Card({ title, children }: { title: string; children: React.ReactNode }) {
  return (
    <section className="border border-gray-200 rounded-xl p-4 bg-white shadow-sm">
      <h2 className="text-lg font-semibold mb-3">{title}</h2>
      {children}
    </section>
  );
}
```

Notice:

- In TSX, you use `className` instead of `class`
    
- Styling is scoped naturally by component boundaries
    

---

## 6) Responsive Design: Tailwind’s Superpower

Tailwind includes built-in responsive prefixes:

- `sm:` small screens
    
- `md:` medium screens
    
- `lg:` large screens
    
- `xl:` extra large
    

Example: one-column on mobile, two-column on larger screens:

```html
<main class="grid gap-4 md:grid-cols-2">
  <section class="border rounded-xl p-4">Today</section>
  <section class="border rounded-xl p-4">Add task</section>
</main>
```

### Explanation

- Default: `grid` with one column
    
- At medium screens and above: `md:grid-cols-2`
    

You get responsiveness without writing media queries.

---

## 7) States: Hover, Focus, Disabled

Tailwind supports interaction states through prefixes:

### Hover

```html
<button class="bg-blue-600 text-white px-4 py-2 rounded-lg hover:bg-blue-700">
  Add
</button>
```

### Focus (keyboard accessibility)

```html
<input class="border rounded-lg p-2 focus:outline-none focus:ring-2 focus:ring-blue-500">
```

### Disabled

```html
<button class="bg-gray-300 text-gray-600 px-4 py-2 rounded-lg disabled:opacity-50" disabled>
  Add
</button>
```

These patterns help you build interfaces that feel polished.

---

## 8) “But Won’t This Make My HTML Messy?”

It can look busy at first. Two reasons it still works well:

1. **Components keep things contained.**  
    Most Tailwind class lists live inside small React components, not in one giant file.
    
2. **You stop writing lots of custom CSS.**  
    Instead of maintaining a big stylesheet and searching for class definitions, you see styles directly where they apply.
    

In practice, you trade:

- fewer CSS files
    
- fewer naming problems
    
- more readable component-level styling
    

---

## 9) When You Still Write Custom CSS

Tailwind covers 95% of everyday styling. But you may still write CSS for:

- complex animations
    
- rare edge cases
    
- third-party library overrides
    
- custom fonts or base styles
    

Tailwind is meant to reduce CSS, not forbid it.

---

## 10) A Small Tailwind Build Example: A Task Item Row

Here’s a simple “task item” UI that looks app-like:

```html
<li class="flex items-center gap-3 border border-gray-200 rounded-lg p-3 bg-white">
  <input type="checkbox" class="h-4 w-4">
  <span class="flex-1 text-gray-900">Finish exercises</span>
  <span class="text-sm text-gray-500">Due: 2026-02-10</span>
  <button class="text-sm px-3 py-1 rounded-md border hover:bg-gray-50">
    Remove
  </button>
</li>
```

### What to notice

- `flex` layout aligns everything in a row
    
- `flex-1` makes the task title take remaining space
    
- spacing is handled by `gap-3` and padding classes
    
- hover effects are quick to add
    

---
