# State Management: Organising Data and UI State in React

In Materials 04 — React and TypeScript you learned **state** with `useState`: data that belongs to a component and can change over time. You saw how state drives the UI and how to lift it to a parent when several components need it.

As apps grow, new questions arise: Where should *shared* state live? How do you handle data from the API? When do you need a global store? This topic introduces **state management** as a broader design concern — how to structure and share state across your React app.

**Prerequisites:** React basics (Materials 04): `useState`, props, component hierarchy, and the idea that state lives in the lowest common ancestor that needs it.

---

## What Is State Management?

**State management** is the practice of deciding:

- **Where** state lives (which component or store).
- **How** it is updated (direct setters, actions, reducers).
- **How** it is shared (props, Context, external library).

React gives you `useState` and `useReducer` out of the box. For many apps that's enough. "State management" becomes a topic when you need to coordinate state across many components or when the data flow gets complex.

**The core idea:** Your UI is a reflection of state. When state changes, React re-renders. The challenge is keeping state in the right place so that:

- Data flows predictably (no contradictions).
- Updates happen in one place (single source of truth).
- You don't pass props through many levels just to reach a deeply nested component (prop drilling).

---

## Local vs Shared State

| Kind | Scope | Typical use |
|------|-------|-------------|
| **Local state** | One component | Form inputs, toggle menus, modal open/closed. Use `useState` in that component. |
| **Lifted state** | Parent and children via props | Tasks list owned by `App`, passed to `TaskList` and `TaskForm`. |
| **Shared / global state** | Many unrelated components | Current user, theme (light/dark), language. Needs Context or an external store. |

**Rule of thumb:** Start with local state. Lift state only when a child needs it. Introduce Context or a global store only when the same data is needed in many parts of the tree without passing it through every level.

### Example: Local state

State that only one component needs stays in that component. No other component reads or updates it.

```tsx
function TaskFormPage() {
  // Form fields: only this page cares about them until submit.
  const [title, setTitle] = useState("");
  const [priority, setPriority] = useState<"low" | "medium" | "high">("medium");
  const [error, setError] = useState("");

  const handleSubmit = async () => {
    // Validate, call API, navigate...
  };

  return (
    <form onSubmit={handleSubmit}>
      <input value={title} onChange={(e) => setTitle(e.target.value)} />
      <select value={priority} onChange={(e) => setPriority(e.target.value)}>
        {/* options */}
      </select>
      {error && <span className="text-error">{error}</span>}
      <button type="submit">Save</button>
    </form>
  );
}
```

### Example: Lifted state

When a parent and its children all need the same data, keep it in the parent and pass it down.

```tsx
function App() {
  // App owns tasks; children receive data and callbacks via props
  const [tasks, setTasks] = useState<Task[]>([]);

  const handleToggle = (id: string) => {
    // Immutable update: create new array with the toggled task
    setTasks((prev) =>
      prev.map((t) => (t.id === id ? { ...t, completed: !t.completed } : t))
    );
  };

  return (
    <Routes>
      <Route path="/" element={<TaskListPage tasks={tasks} onToggle={handleToggle} />} />
      <Route path="/add" element={<TaskFormPage onAdd={(t) => setTasks((prev) => [...prev, t])} />} />
    </Routes>
  );
}
```

### Prop drilling

When you lift state high in the tree, you may end up passing props through components that don't use them — only to reach a deep child. That's **prop drilling**:

```tsx
// App passes user to Layout, which passes it to Header, which passes it to ProfileButton.
// Layout and Header don't use user; they just forward it.
<App user={user}>
  <Layout user={user}>
    <Header user={user}>
      <ProfileButton user={user} />
    </Header>
  </Layout>
</App>
```

When prop drilling gets annoying, Context or a global store can avoid it. For a few levels, passing props is usually fine.

---

## Server State vs Client State

An important distinction in fullstack apps:

| Type | Source | Examples |
|------|--------|----------|
| **Server state** | Data from the API (backend/database) | Tasks, user profile, categories. |
| **Client state** | Stored only in the browser | Form drafts, UI preferences (sidebar open), selected tab. |

**Why it matters:**

- **Server state** is the "source of truth" on the backend. The frontend fetches it, may cache it, and refetches when needed. It can become stale if the backend or another user changes data.
- **Client state** exists only in the frontend. It doesn't need to be synced with the server (unless you persist it, e.g. to localStorage).

Many beginners put fetched data in component state and treat it like any other state. That works, but you then need to decide: when to refetch, how to handle loading and errors, and how to avoid duplicate requests. Libraries like **React Query** (TanStack Query) specialise in server state: caching, refetching, and invalidation. For smaller apps, `useState` + `useEffect` + your API client is often enough.

### Example: Server state with useState + useEffect

A typical pattern for fetching data when a page mounts:

```tsx
function TasksListPage() {
  const [tasks, setTasks] = useState<Task[]>([]);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState<string | null>(null);

  useEffect(() => {
    let cancelled = false;  // Prevents state update after unmount (e.g. user navigates away quickly)

    async function load() {
      setLoading(true);
      setError(null);
      try {
        const data = await getTasks();  // API call
        if (!cancelled) setTasks(data);  // Only update if still mounted
      } catch (e) {
        if (!cancelled) setError("Failed to load tasks");
      } finally {
        if (!cancelled) setLoading(false);
      }
    }

    load();
    return () => { cancelled = true; };  // Cleanup: ignore result if component unmounts
  }, []);  // Empty deps: run once on mount

  if (loading) return <p>Loading...</p>;
  if (error) return <p>{error}</p>;
  return <ul>{tasks.map((t) => <TaskItem key={t.id} task={t} />)}</ul>;
}
```

**When to refetch:** After a mutation (e.g. add task, toggle completion), call the API again and update state. Or refetch when the user navigates back to the page. React Query automates this with `refetch`, `invalidateQueries`, and cache keys.

### Example: Client state

UI-only state that never goes to the server:

```tsx
const [sidebarOpen, setSidebarOpen] = useState(false);
const [selectedTab, setSelectedTab] = useState<"all" | "active">("all");
const [searchQuery, setSearchQuery] = useState("");
```

---

## Common Patterns in This Course

### 1. Lifted state (e.g. Exercise 2)

Tasks live in `App`. `App` owns `tasks` and passes them (and handlers like `onToggle`, `onAdd`) down to pages via props. Pages don't fetch; they receive data and callbacks.

**Pros:** Simple, explicit, easy to follow; single source of truth in `App`.  
**Cons:** Prop drilling if the tree gets deep; `App` re-renders on every task change.

**When it fits:** Small to medium apps where the task list is used in a few places and the hierarchy is shallow. Exercise 2 uses this with localStorage as persistence.

### 2. Per-page data fetching (e.g. Exercise 3)

Each page fetches the data it needs. `TasksListPage` fetches the list; `TaskDetailsPage` fetches one task by id. No shared task state in `App`. When the user navigates away and back, the page refetches (or you could cache).

**Pros:** No prop drilling; each page is self-contained; data is fresh when you navigate; `App` stays thin (just routes).  
**Cons:** You may refetch when navigating back; no single frontend cache unless you add one.

**Example:** `App` only renders routes; each page owns its own fetch logic:

```tsx
// App only renders routes; no shared task state
function App() {
  return (
    <Routes>
      <Route path="/" element={<TasksListPage />} />
      <Route path="/tasks/:id" element={<TaskDetailsPage />} />
      <Route path="/tasks/new" element={<TaskFormPage />} />
    </Routes>
  );
}

// TasksListPage fetches its own data; no props from App
function TasksListPage() {
  const [tasks, setTasks] = useState<Task[]>([]);
  useEffect(() => {
    getTasks().then(setTasks);  // Fetch on mount
  }, []);
  const handleToggle = async (id: number) => {
    await toggleTask(id);  // Call API
    setTasks((prev) => prev.map((t) => (t.id === id ? { ...t, completed: !t.completed } : t)));  // Update local state
  };
  return (/* ... */);
}
```

### 3. Context for shared data (e.g. auth, theme)

When many components need the same data (current user, theme) and passing props through every level is painful, use **React Context**. You wrap part of the tree in a Provider; any descendant can `useContext` to read (and optionally update) the value.

**Example: Auth context**

```tsx
type AuthContextType = {
  user: User | null;
  login: (email: string, password: string) => Promise<void>;
  logout: () => void;
};

const AuthContext = createContext<AuthContextType | null>(null);

export function AuthProvider({ children }: { children: React.ReactNode }) {
  const [user, setUser] = useState<User | null>(null);

  const login = async (email: string, password: string) => {
    const data = await loginApi(email, password);
    setUser(data.user);  // Update context; all consumers re-render
  };

  const logout = () => {
    setUser(null);
  };

  return (
    <AuthContext.Provider value={{ user, login, logout }}>
      {children}
    </AuthContext.Provider>
  );
}

export function useAuth() {
  const ctx = useContext(AuthContext);
  if (!ctx) throw new Error("useAuth must be used within AuthProvider");
  return ctx;
}

// Any component, anywhere in the tree, can read auth without prop drilling:
function ProfileButton() {
  const { user, logout } = useAuth();
  return user ? (
    <button onClick={logout}>{user.name}</button>
  ) : (
    <Link to="/login">Log in</Link>
  );
}
```

**When to use Context:** Auth state, theme, language, or other "global" UI concerns. Use it sparingly; it doesn't replace local state or prop passing for data that only a few components need. Avoid putting frequently changing data (e.g. a large task list) in Context — every context change re-renders all consumers.

---

## Choosing an Approach

| Situation | Suggested approach |
|-----------|--------------------|
| Form fields, modal open/closed | `useState` in the component that uses it |
| Data needed by a parent and a few children | Lift state to the parent; pass via props |
| Data needed by many unrelated components | Context or a small global store |
| Data from the API, one page | `useState` + `useEffect` + fetch in that page |
| Data from the API, shared across pages | Per-page fetch with refetch, or a client cache (e.g. React Query) |
| Complex global state, many updates | Redux, Zustand, or similar (beyond this course’s scope) |
| Filtered or computed list from existing state | Derive with a variable or `useMemo`; don't store separately |

---

## Common Pitfalls

1. **Storing derived data** — If `filteredTasks` can be computed from `tasks` and `filter`, don't store it. Compute it.
2. **Putting everything in Context** — Context is for truly shared data. Don't put the task list in Context if only 1–2 pages need it; prop drilling a couple of levels is fine.
3. **Syncing state from props** — Avoid `useState(prop)` and then `useEffect` to sync when the prop changes. Prefer either controlled (parent owns it, pass value + onChange) or uncontrolled with a key to reset.
4. **Forgetting to handle loading/error** — Server state needs `loading` and `error` states. Show a spinner or skeleton while loading; show a message and retry option on error.

---

## Derived State: Compute Don't Store

Not everything needs to be state. If a value can be **computed** from existing state or props, compute it instead of storing it. This avoids sync bugs and keeps your state minimal.

```tsx
// ✅ Good: derive filtered list from tasks + filter (single source of truth)
const [tasks, setTasks] = useState<Task[]>([]);
const [filter, setFilter] = useState<"all" | "active">("all");
const filteredTasks = filter === "all" ? tasks : tasks.filter((t) => !t.completed);

// ❌ Avoid: storing filteredTasks in state — it can get out of sync with tasks
const [filteredTasks, setFilteredTasks] = useState<Task[]>([]);
useEffect(() => {
  setFilteredTasks(filter === "all" ? tasks : tasks.filter((t) => !t.completed));
}, [tasks, filter]);
```

For expensive computations, use `useMemo` so the result is only recomputed when dependencies change:

```tsx
// Recomputed only when tasks changes; avoids recalculating on every render
const completedCount = useMemo(() => tasks.filter((t) => t.completed).length, [tasks]);
```

---

## Single Source of Truth

A useful principle: **avoid duplicating the same logical data in multiple places**. If the task list is stored in `App` and also in a page, they can get out of sync. Pick one place as the source of truth:

- **Lifted state:** `App` (or a parent) is the source of truth; children receive it via props.
- **Per-page fetch:** The API is the source of truth; each page fetches when it mounts.
- **Context:** The Provider holds the source of truth; consumers read from it.

**What goes wrong with multiple sources:**

```tsx
// ❌ App has tasks; TasksListPage also fetches and keeps its own copy.
// User toggles a task in the list — TasksListPage updates. But App still has the old data.
// If another part of the app reads from App, it sees stale data.
```

---

## Summary: Key Concepts

| Concept | Meaning |
|---------|---------|
| **Local state** | State in one component; others don't need it |
| **Lifted state** | State in a parent; passed to children via props |
| **Shared / global state** | State needed by many components; often via Context |
| **Prop drilling** | Passing props through components that don't use them, just to reach a deep child |
| **Server state** | Data from the API; the backend is the source of truth |
| **Client state** | UI-only data; lives in the frontend |
| **Derived state** | Values computed from other state or props; don't store, compute |
| **Single source of truth** | One place holds the data; others read or derive from it |

---

## What You Should Understand After This Topic

By the end, you should be able to:

- Explain the difference between local, lifted, and shared state.
- Distinguish server state (API data) from client state (UI-only).
- Describe when to lift state vs when to use Context.
- Explain the per-page data fetching pattern and when it fits.
- Apply the "single source of truth" principle when designing state.
- Prefer deriving values from state instead of storing them separately (derived state).
- Recognise prop drilling and when Context might help.
- Implement a basic fetch pattern with loading and error states.
- Know that for most course-sized apps, `useState`, props, and optionally Context are sufficient; larger apps may use libraries like Redux or React Query.
