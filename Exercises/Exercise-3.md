# Exercise 3: Task Tracker — FastAPI Backend and Frontend Integration

## Important!

Implement your solution inside your classroom exercises repository:

- Create a new folder **Exercise-3** in the repo and put the **FastAPI backend** inside `Exercise-3/backend`.
- Use the **example Task Tracker frontend** (or your own Exercise 2 solution) as the frontend. Create new `Exercie-3/frontend` folder and copy the frontend implementation inside the folder. You will **modify** that project to call the new backend instead of localStorage. (Technically it would be possible to use the existing Exercise 2 frontend solution directly, but this way we can keep the history of your progress intact (could use commits also))

**This exercise is based on the example implementation** (e.g. the provided `task-tracker-example` solution). The instructions use the following structure:

```
task-tracker-example/   (or your task-tracker-react project)
  src/
    App.tsx                    ← routes only; no task state or context
    index.css (or styles/)     ← Tailwind + DaisyUI: @import "tailwindcss"; @plugin "daisyui" (Step 5)
    api/
      config.ts                ← getApiBaseUrl (Step 6)
      tasksApi.ts              ← getTasks, getTask, createTask, updateTask, deleteTask (Step 7)
    domain/
      task.ts                  ← Task, Priority
    storage/
      tasksStorage.ts          ← replaced by API; no longer used for persistence
    pages/
      TasksListPage.tsx        ← fetches getTasks() on mount; refetches after toggle/delete
      TaskDetailsPage.tsx      ← fetches getTask(id) on mount; refetches after toggle; navigate after delete
      TaskFormPage.tsx         ← create: no fetch; edit: fetches getTask(id); calls API then navigate
    components/
      Layout.tsx
  main.tsx
  vite.config.ts               ← plugins: [react(), tailwindcss()] (Step 5)
  package.json                 ← tailwindcss, @tailwindcss/vite, daisyui (Step 5)
```

The instructions below refer to these files and give concrete steps to update the example (or your equivalent) so the frontend uses the FastAPI backend.

---

## Overall goal

1. **Set up a Python FastAPI backend** that exposes a REST API for tasks (list, get one, create, update, delete), matching the example frontend’s **Task** type.
2. **Connect the React frontend** to the backend: add an API client and environment variable, then replace `loadTasks` / `saveTasks` and the existing handlers in `App.tsx` with API calls.

By the end, the Task Tracker will be full-stack: the browser talks to your backend, and the backend holds the data in memory (you can add a database later).

The **Python backend** is structured using **layered architecture** (see Materials 08 — Backend Introduction):

- **Routes (API layer)** — Receive HTTP requests, call the service, return HTTP responses. Each resource has its own **route file** (e.g. `routes/tasks.py`), similar to .NET controllers in separate controller classes; `main.py` creates the app and includes the routers.
- **Service layer** — Business logic and use cases; calls the data layer. Lives in `services/task_service.py`.
- **Repository (data access layer)** — In-memory list and CRUD operations; no HTTP. Lives in `store.py` (or `repositories/`). Later you can replace this with a database without changing the service or routes.

Request flow: **Routes → Service → Repository** (then response back up).

Backend folder structure when you’re done:

```
Exercise-3/backend/
  main.py                    ← creates FastAPI app, CORS, includes routers; root route only
  models.py                  ← Task, TaskCreate, TaskUpdate, Priority
  store.py                   ← repository (data access): in-memory list + CRUD (temporary replacement for database)
  routes/
    tasks.py                 ← task endpoints (like a .NET TasksController); calls service
  services/
    task_service.py          ← service layer: list_tasks, get_task, create_task, update_task, delete_task
  venv/
```

---

## What you need before starting

- The **example frontend** (e.g. `task-tracker-example`) or your own Exercise 2 Task Tracker. The example uses:
  - **Task** in `src/domain/task.ts`: `id`, `title`, `completed`, `priority`, `createdAt`, `updatedAt?`
  - **Priority**: `"low" | "medium" | "high"`
  - **App.tsx**: `useState<Task[]>(() => loadTasks())`, `useEffect` → `saveTasks(tasks)`, and handlers `addTask`, `deleteTask`, `toggleCompleted`, `updateTask`
  - **TaskFormPage**: in create mode builds a `Task` with `crypto.randomUUID()` and `Date.now()` and calls `onAdd(newTask)`; in edit mode calls `onUpdate(id, { title, priority })`
- **Python 3.10+** installed (`python --version` or `python3 --version`).

---

# Step 1 — Create the FastAPI project (backend folder and dependencies)

### Goal of this step

Create a dedicated folder for the backend and install FastAPI and a few dependencies. By the end you will have a runnable FastAPI app that returns a simple JSON response.

---

### Create the backend folder

In your **Exercise-3** folder, create a `backend` directory:

```
Exercise-3/
  backend/
```

All backend code (Python) will go under `backend/`.

---

### Create a virtual environment

> The virtual enviroment is not mandatory, so if you have problems with activating the virtual enviroment, you can just skip to the install dependencies step

From the `backend` folder:

```bash
cd backend
python -m venv venv
```

Activate it:

- **Windows (PowerShell):** `.\venv\Scripts\Activate.ps1`
- **Windows (cmd):** `venv\Scripts\activate.bat`
- **macOS/Linux:** `source venv/bin/activate`

---

### Install dependencies

```bash
pip install fastapi uvicorn
```

- **FastAPI**: web framework for defining routes and request/response models.
- **uvicorn**: ASGI server that runs your FastAPI app.

---

### Minimal app to verify setup

Create `backend/main.py`:

```python
from fastapi import FastAPI

app = FastAPI(title="Task Tracker API")

@app.get("/")
def root():
    return {"message": "Task Tracker API is running"}
```

Run the server:

```bash
uvicorn main:app --reload
```

Open **http://127.0.0.1:8000** in the browser. You should see `{"message":"Task Tracker API is running"}`.

Optional: open **http://127.0.0.1:8000/docs** to see automatic API documentation.

---

### When to move on

- You have a `backend` folder with `venv` and `main.py`.
- `uvicorn main:app --reload` runs without errors and the root URL returns JSON.

---

# Step 2 — Task model and repository (data access layer)

### Goal of this step

Define a **Pydantic** model for Task that matches your frontend’s Task type, and implement the **repository (data access) layer**: an in-memory store with functions to get/add/update/delete tasks. By the end you will have:

- A `Task` model (and request body model for create/update) in Python.
- A **repository** (`store.py`) that holds the in-memory list and exposes: get all, get by id, create, update, delete. No HTTP — this layer only talks to “storage”; later you can replace it with a database.

---

### Pydantic model for Task

Create `backend/models.py` (or put this in `main.py` if you prefer a single file for now).

Your frontend uses:

- `id`: string
- `title`: string
- `completed`: boolean
- `priority`: "low" | "medium" | "high"
- `createdAt`: number
- `updatedAt`: optional number

In Python you can use a Pydantic model and an enum for priority:

```python
from enum import Enum
from pydantic import BaseModel
from typing import Optional

class Priority(str, Enum):
    low = "low"
    medium = "medium"
    high = "high"

class Task(BaseModel):
    id: str
    title: str
    completed: bool = False
    priority: Priority = Priority.medium
    createdAt: int  # timestamp in milliseconds (same as frontend Date.now())
    updatedAt: Optional[int] = None

# Request body for creating a task (id and timestamps are set by backend).
class TaskCreate(BaseModel):
    title: str
    completed: bool = False
    priority: Priority = Priority.medium

# Request body for PATCH: only include fields that change.
class TaskUpdate(BaseModel):
    title: Optional[str] = None
    completed: Optional[bool] = None
    priority: Optional[Priority] = None
```

Use **camelCase** in the model (`createdAt`, `updatedAt`) so that FastAPI serializes JSON that matches your frontend. By default Pydantic uses the field names as-is, so your frontend’s `task.createdAt` will match.

---

### Repository (data access layer): in-memory store

Create `backend/store.py` as the **repository** — the data access layer that talks to “storage” (here, an in-memory list). Routes and business logic will not call this directly; they will use the service layer (Step 3).

```python
import time
import uuid
from models import Task, TaskCreate, TaskUpdate, Priority

# In-memory list (resets when the server restarts)
_tasks: list[Task] = []

# Return current time in milliseconds (same as JavaScript Date.now()).
def _now() -> int:
    return int(time.time() * 1000)

def get_all_tasks() -> list[Task]:
    return list(_tasks)

def get_task_by_id(task_id: str) -> Task | None:
    for t in _tasks:
        if t.id == task_id:
            return t
    return None

def create_task(body: TaskCreate) -> Task:
    task_id = str(uuid.uuid4())
    now = _now()
    task = Task(
        id=task_id,
        title=body.title,
        completed=body.completed,
        priority=body.priority,
        createdAt=now,
        updatedAt=None,
    )
    _tasks.append(task)
    return task

def update_task(task_id: str, body: TaskUpdate) -> Task | None:
    task = get_task_by_id(task_id)
    if not task:
        return None
    # Only apply fields that were sent in the request
    updates = body.model_dump(exclude_unset=True)
    updates["updatedAt"] = _now()
    updated = task.model_copy(update=updates)
    idx = next(i for i, t in enumerate(_tasks) if t.id == task_id)
    _tasks[idx] = updated
    return updated

def delete_task(task_id: str) -> bool:
    global _tasks
    before = len(_tasks)
    _tasks = [t for t in _tasks if t.id != task_id]
    return len(_tasks) < before
```

If your frontend uses different field names (e.g. `dueDate` instead of `priority`), adjust the models and store to match.

---

### When to move on

- You have `Task`, `TaskCreate`, and `TaskUpdate` defined.
- You have the repository (`store.py`) with get_all, get_by_id, create, update, delete.
- The app still runs with `uvicorn main:app --reload` (you haven’t broken imports).

---

# Step 3 — Service layer (backend)

### Goal of this step

Add a **service layer** between the API (routes) and the repository. The service will contain use-case functions that the routes will call; it will use the repository for data access. For this exercise the service mostly delegates to the repository, but the pattern makes it easy to add validation or business rules later.

---

### Create the task service

Create a `services` folder and `backend/services/task_service.py`:

```python
from models import Task, TaskCreate, TaskUpdate
from store import (
    get_all_tasks as repo_get_all,
    get_task_by_id as repo_get_by_id,
    create_task as repo_create,
    update_task as repo_update,
    delete_task as repo_delete,
)

def list_tasks() -> list[Task]:
    return repo_get_all()

def get_task(task_id: str) -> Task | None:
    return repo_get_by_id(task_id)

def create_task(body: TaskCreate) -> Task:
    return repo_create(body)

def update_task(task_id: str, body: TaskUpdate) -> Task | None:
    return repo_update(task_id, body)

def delete_task(task_id: str) -> bool:
    return repo_delete(task_id)
```

You can add business logic here later (e.g. “title must not be empty”, “only owner can delete”). For now the service simply delegates to the repository.

---

### When to move on

- You have `backend/services/task_service.py` with `list_tasks`, `get_task`, `create_task`, `update_task`, `delete_task` that call the repository.
- No HTTP or FastAPI imports in the service — it stays independent of the web layer.

---

# Step 4 — Implement the REST API endpoints (backend)

### Goal of this step

Wire your FastAPI **routes** to the **service layer** (not directly to the repository). The routes handle HTTP; the service handles use cases and calls the repository. The frontend will then be able to:

- **GET /api/tasks** — list all tasks
- **GET /api/tasks/{id}** — get one task
- **POST /api/tasks** — create a task
- **PATCH /api/tasks/{id}** — update a task (partial)
- **DELETE /api/tasks/{id}** — delete a task

Use the same path prefix `/api/tasks` so the frontend can use a single base URL (e.g. `http://localhost:8000/api`).

---

Put task endpoints in a **dedicated route file** (similar to a .NET controller in its own file). Then **register the router** in `main.py`. This keeps `main.py` small and groups all task-related endpoints in one place.

---

### 4.1 Create the tasks router — `routes/tasks.py`

Create a `routes` folder and `backend/routes/tasks.py`. Use FastAPI’s **APIRouter** so you can mount it under a prefix in `main.py`:

```python
from fastapi import APIRouter, HTTPException

from models import Task, TaskCreate, TaskUpdate
from services.task_service import (
    list_tasks as service_list_tasks,
    get_task as service_get_task,
    create_task as service_create_task,
    update_task as service_update_task,
    delete_task as service_delete_task,
)

router = APIRouter(prefix="/tasks", tags=["tasks"])

@router.get("", response_model=list[Task])
def list_tasks():
    return service_list_tasks()

@router.get("/{task_id}", response_model=Task)
def get_task(task_id: str):
    task = service_get_task(task_id)
    if not task:
        raise HTTPException(status_code=404, detail="Task not found")
    return task

@router.post("", response_model=Task, status_code=201)
def post_task(body: TaskCreate):
    return service_create_task(body)

@router.patch("/{task_id}", response_model=Task)
def patch_task(task_id: str, body: TaskUpdate):
    task = service_update_task(task_id, body)
    if not task:
        raise HTTPException(status_code=404, detail="Task not found")
    return task

@router.delete("/{task_id}", status_code=204)
def remove_task(task_id: str):
    if not service_delete_task(task_id):
        raise HTTPException(status_code=404, detail="Task not found")
    return None
```

The router uses `prefix="/tasks"`; you will mount it with prefix `/api` in `main.py`, so the full paths are `/api/tasks`, `/api/tasks/{id}`, etc.

---

### 4.2 Update `main.py`: app, CORS, and include the router

Keep **main.py** focused on creating the app, adding CORS, and **including** the route modules (like registering controllers in .NET). Move the root route here; all task routes live in `routes/tasks.py`:

```python
from fastapi import FastAPI
from fastapi.middleware.cors import CORSMiddleware

from routes.tasks import router as tasks_router

app = FastAPI(title="Task Tracker API")

app.add_middleware(
    CORSMiddleware,
    allow_origins=["http://localhost:5173", "http://127.0.0.1:5173"],
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
)

@app.get("/")
def root():
    return {"message": "Task Tracker API is running"}

app.include_router(tasks_router, prefix="/api")
```

Flow: **request → route (routes/tasks.py) → service → repository** (store). The route turns service results (or “not found”) into HTTP status codes and JSON.

---

### API contract summary (for frontend)

| Method | Path            | Request body                        | Response / status |
| ------ | --------------- | ----------------------------------- | ----------------- |
| GET    | /api/tasks      | —                                   | 200, `Task[]`     |
| GET    | /api/tasks/{id} | —                                   | 200 `Task` or 404 |
| POST   | /api/tasks      | `{ title, completed?, priority? }`  | 201 `Task`        |
| PATCH  | /api/tasks/{id} | `{ title?, completed?, priority? }` | 200 `Task` or 404 |
| DELETE | /api/tasks/{id} | —                                   | 204 or 404        |

---

### Test the API manually

1. Start the backend: `uvicorn main:app --reload`.
2. Open **http://127.0.0.1:8000/docs** and try:
   - **POST /api/tasks** with body `{"title": "First task", "priority": "high"}` → should return 201 and a task with `id`, `createdAt`, etc.
   - **GET /api/tasks** → should return a list containing that task.
   - **PATCH /api/tasks/{id}** with `{"completed": true}` → task should show `completed: true`.
   - **DELETE /api/tasks/{id}** → 204, then GET list should no longer include it.

---

### When to move on

- All five endpoints work and return the expected status codes and JSON.
- Task endpoints live in **routes/tasks.py** (APIRouter); **main.py** only creates the app, adds CORS, and includes the router with `prefix="/api"`.
- Routes call the **service**; the service uses the **repository** (store). No direct store imports in the route file or main.
- CORS is enabled so that a request from `http://localhost:5173` would be allowed (you’ll confirm in Step 7 with the real frontend).

---

# Step 5 — Frontend: Tailwind CSS 4 and DaisyUI

### Goal of this step

Add **Tailwind CSS v4** and **DaisyUI** to the React (Vite) frontend project so you can style the app with utility classes and DaisyUI’s component classes. Do this in the **frontend project root** (same folder as `package.json` and `vite.config.ts`).

---

### 5.1 Install Tailwind CSS 4 and the Vite plugin

From the frontend project root:

```bash
npm install tailwindcss @tailwindcss/vite
```

Tailwind v4 uses a first-party Vite plugin; you do not need PostCSS or a separate `tailwind.config.js` for basic setup.

---

### 5.2 Install DaisyUI

DaisyUI v4 works with Tailwind CSS 4. Install it as a dev dependency:

```bash
npm install -D daisyui@latest
```

Use `daisyui@latest` or check [DaisyUI’s install docs](https://v4.daisyui.com/docs/install) for the version that matches your Tailwind major version.

---

### 5.3 Enable Tailwind in Vite

In **vite.config.ts**, import the Tailwind plugin and add it to the `plugins` array (together with React or any existing plugins):

```ts
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';
import tailwindcss from '@tailwindcss/vite';

export default defineConfig({
  plugins: [react(), tailwindcss()],
  // ... rest of your config (e.g. resolve.alias)
});
```

---

### 5.4 Add Tailwind and DaisyUI to your CSS

In your main CSS file (e.g. **src/index.css** or **src/styles/app.css** — the one you import from `main.tsx`), add:

```css
@import 'tailwindcss';
@plugin "daisyui";
```

If the file already has other rules, keep them; the `@import` and `@plugin` lines should be at the top so Tailwind and DaisyUI are available everywhere.

**Optional — DaisyUI themes:** You can pick a default theme or enable light/dark via `data-theme`:

```css
@import 'tailwindcss';
@plugin "daisyui" {
  themes: light --default, dark --prefersdark;
}
```

Then use `data-theme="light"` or `data-theme="dark"` on `<html>` or a wrapper, or rely on `--prefersdark` for system preference.

---

### 5.5 Using Tailwind and DaisyUI in components

- **Tailwind:** Use utility classes in `className` for layout, spacing, typography, and colors, e.g. `flex`, `gap-4`, `p-4`, `text-lg`, `rounded-lg`, `bg-base-200`.
- **DaisyUI:** Use component classes for buttons, cards, forms, and more. Examples:
  - Buttons: `className="btn btn-primary"`, `btn-secondary`, `btn-error`, `btn-sm`, `btn-outline`
  - Cards: `className="card bg-base-100 shadow-xl"` with children `card-body`, `card-title`
  - Inputs: `className="input input-bordered"`, `input-primary`, `input-sm`
  - Badges: `className="badge badge-primary"`, `badge-success`, `badge-warning`
  - Alerts: `className="alert alert-info"`, `alert-warning`, `alert-error`

You can combine both: `className="card bg-base-100 shadow-xl p-4 rounded-lg"`. See [Tailwind CSS docs](https://tailwindcss.com/docs) and [DaisyUI components](https://daisyui.com/components/) for more.

---

### When to move on

- `tailwindcss` and `@tailwindcss/vite` are installed; `vite.config.ts` includes the Tailwind plugin.
- `daisyui` is installed; your main CSS file has `@import "tailwindcss";` and `@plugin "daisyui";`.
- The dev server runs without CSS errors; you can use Tailwind and DaisyUI classes in any component (e.g. add `className="btn btn-primary"` to a button and see the styled result).

---

# Step 6 — Frontend: environment variable and API base URL

### Goal of this step

In the **example Task Tracker project** (e.g. `task-tracker-react-vite-ts`), add an environment variable for the backend URL and a small config module. All API requests will use this base URL.

---

### 6.1 Add `.env` in the React project root

In the project root (same folder as `package.json` and `vite.config.ts`), create a file named `.env`:

```env
VITE_API_BASE_URL=http://localhost:8000
```

In Vite, only variables prefixed with `VITE_` are exposed to the client. Do not add a trailing slash.

---

### 6.2 Create the API config module

Create a new folder `src/api` and inside it create `src/api/config.ts`:

```ts
const baseUrl = import.meta.env.VITE_API_BASE_URL ?? 'http://localhost:8000';

export function getApiBaseUrl(): string {
  return baseUrl.replace(/\/$/, '');
}
```

You will use `getApiBaseUrl()` in the next step when building request URLs.

---

### When to move on

- `.env` exists in the project root with `VITE_API_BASE_URL=http://localhost:8000`.
- `src/api/config.ts` exists and exports `getApiBaseUrl`.
- Restart the Vite dev server after adding or changing `.env` so the value is picked up.

---

# Step 7 — Frontend: API client for tasks

### Goal of this step

Add a **tasks API client** in the example project: one module that implements `getTasks`, `getTask(id)`, `createTask`, `updateTask`, and `deleteTask` using `fetch`. This matches the backend API you built in Steps 1–4. The app will use this module instead of `tasksStorage` for loading and saving.

---

### Create `src/api/tasksApi.ts`

Create a new file `src/api/tasksApi.ts`. Use the existing **Task** type from `src/domain/task.ts` (same as in the example):

```ts
import type { Task } from '../domain/task';
import { getApiBaseUrl } from './config';

const base = () => getApiBaseUrl();

export async function getTasks(): Promise<Task[]> {
  const res = await fetch(`${base()}/api/tasks`);
  if (!res.ok) throw new Error('Failed to fetch tasks');
  return res.json();
}

export async function getTask(id: string): Promise<Task | null> {
  const res = await fetch(`${base()}/api/tasks/${id}`);
  if (res.status === 404) return null;
  if (!res.ok) throw new Error('Failed to fetch task');
  return res.json();
}

export async function createTask(body: {
  title: string;
  priority?: Task['priority'];
}): Promise<Task> {
  const res = await fetch(`${base()}/api/tasks`, {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({
      title: body.title.trim(),
      completed: false,
      priority: body.priority ?? 'medium',
    }),
  });
  if (!res.ok) throw new Error('Failed to create task');
  return res.json();
}

export async function updateTask(
  id: string,
  patch: { title?: string; completed?: boolean; priority?: Task['priority'] }
): Promise<Task> {
  const res = await fetch(`${base()}/api/tasks/${id}`, {
    method: 'PATCH',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify(patch),
  });
  if (res.status === 404) throw new Error('Task not found');
  if (!res.ok) throw new Error('Failed to update task');
  return res.json();
}

export async function deleteTask(id: string): Promise<void> {
  const res = await fetch(`${base()}/api/tasks/${id}`, { method: 'DELETE' });
  if (res.status === 404) throw new Error('Task not found');
  if (!res.ok) throw new Error('Failed to delete task');
}
```

The backend returns JSON with camelCase (`createdAt`, `updatedAt`) to match the frontend `Task` type.

---

### When to move on

- `src/api/tasksApi.ts` exists and exports the five functions above.
- It does not import or use `tasksStorage`.

---

# Step 8 — Frontend: Each page fetches its own data

### Goal of this step

Update the example implementation so that:

- **App.tsx** only renders routes; it holds **no task state** and no context. Remove all task-related state, effects, and handlers from App.
- **Each page is responsible for fetching the data it needs:** TasksListPage fetches the list, TaskDetailsPage fetches the single task by id, TaskFormPage in edit mode fetches the task to edit. After mutations (create, update, delete, toggle), the page either refetches its data or navigates so the user sees up-to-date content.

**Full reference:** The **frontend example solution** in `Exercise-3/frontend-example-solution/` implements this pattern (no context): App renders routes only; TasksListPage fetches `getTasks()` and refetches after toggle/delete; TaskDetailsPage fetches `getTask(id)` and refetches or navigates; TaskFormPage in create mode has no fetch and navigates after create, in edit mode fetches `getTask(id)` then navigates after update.

---

### 8.1 Update `src/App.tsx`

Remove task state, storage/API loading, and any task handlers. App only renders routes.

**Remove** `import { loadTasks, saveTasks } from './storage/tasksStorage';` and any `Task` / `getTasks` / context imports.

**Replace** the entire component body with just the routes (no `useState`, no `useEffect`, no Provider):

```tsx
import { Navigate, Route, Routes } from 'react-router-dom';

import Layout from './components/Layout';
import TasksListPage from './pages/TasksListPage';
import TaskDetailsPage from './pages/TaskDetailsPage';
import TaskFormPage from './pages/TaskFormPage';
import NotFoundPage from './pages/NotFoundPage';

export default function App() {
  return (
    <Routes>
      <Route path="/" element={<Layout />}>
        <Route index element={<Navigate to="/tasks" replace />} />
        <Route path="tasks" element={<TasksListPage />} />
        <Route path="tasks/new" element={<TaskFormPage mode="create" />} />
        <Route path="tasks/:id" element={<TaskDetailsPage />} />
        <Route path="tasks/:id/edit" element={<TaskFormPage mode="edit" />} />
        <Route path="*" element={<NotFoundPage />} />
      </Route>
    </Routes>
  );
}
```

---

### 8.2 Update `src/pages/TasksListPage.tsx`

The list page **fetches the task list itself** on mount and **refetches** after toggle or delete.

#### 8.2.1 Local state and refetch

**Remove** any props. **Add** local state `const [tasks, setTasks] = useState<Task[]>([]);`, `useEffect(() => { getTasks().then(setTasks).catch(...); }, []);`, and `const refetch = () => getTasks().then(setTasks);`. **Imports:** `useState`, `useEffect`, `Task` from domain, `getTasks`, `updateTask`, `deleteTask` from `../api/tasksApi`.

Sort for display: `const tasksSorted = [...tasks].sort((a, b) => (b.createdAt || 0) - (a.createdAt || 0));` and render `tasksSorted`. On toggle: `updateTask(task.id, { completed: !task.completed }).then(refetch)`. On delete: `deleteTask(task.id).then(refetch)`.

#### 8.2.2 Toggle and delete by calling the API then refresh

**Replace** the place where you pass `onToggle` to the checkbox (e.g. `onChange={() => onToggle(task.id)}`) **with** a handler that calls the API and then refreshes:

```ts
onChange={() => {
  updateTask(task.id, { completed: !task.completed }).then(refreshTasks);
}}
```

**Replace** the delete button’s `onClick` (where you currently call `onDelete(task.id)`) **with:**

```ts
onClick={() => {
  const ok = confirm(`Delete "${task.title}"?`);
  if (ok) deleteTask(task.id).then(refreshTasks);
}}
```

So the component no longer receives `onToggle` or `onDelete`; it uses `updateTask` and `deleteTask` from `tasksApi` and then `refreshTasks()`.

---

### 8.3 Update `src/pages/TaskDetailsPage.tsx`

The details page also uses context and the API.

#### 8.3.1 Use context instead of props

**Remove** the props type and props. **Add:**

```ts
import { useTasks } from '../context/TasksContext';
import { updateTask, deleteTask } from '../api/tasksApi';
```

At the top of the component: `const { tasks, refreshTasks } = useTasks();` and keep `const { id } = useParams();` and `const task = tasks.find(t => t.id === id);`.

#### 8.3.2 Toggle and delete by calling the API then refresh

For the “Mark as completed” / “Mark as active” button, **replace** the existing `onClick` **with:**

```ts
onClick={() => {
  if (!task) return;
  updateTask(task.id, { completed: !task.completed }).then(refreshTasks);
}}
```

For the Delete button, **replace** the existing handler **with:**

```ts
onClick={() => {
  const ok = confirm(`Delete "${task.title}"?`);
  if (!ok) return;
  deleteTask(task.id).then(() => {
    refreshTasks();
    navigate('/tasks');
  });
}}
```

---

### 8.4 Update `src/pages/TaskFormPage.tsx`

The form page uses context to read `tasks` (for edit mode) and `refreshTasks`, and calls the API for create and update.

#### 8.4.1 Use context and remove callback props

**Change the Props type** so the page only receives `mode` (no `tasks`, `onAdd`, `onUpdate`), and update the component signature to destructure only `mode`:

```ts
type Props = {
  mode: 'create' | 'edit';
};

export default function TaskFormPage({ mode }: Props) {
```

**Add imports:**

```ts
import { useTasks } from '../context/TasksContext';
import { createTask, updateTask } from '../api/tasksApi';
```

At the top of the component: `const { tasks, refreshTasks } = useTasks();` and keep `const { id } = useParams();`, `const taskToEdit = mode === 'edit' ? tasks.find(t => t.id === id) : undefined`, and the rest of the existing state.

#### 8.4.2 Create and edit by calling the API then refresh and navigate

**Replace** the entire `handleSubmit` body so that:

- **Create mode:** call `createTask({ title: trimmed, priority })`, then `refreshTasks()`, then `navigate('/tasks')`.
- **Edit mode:** call `updateTask(taskToEdit!.id, { title: trimmed, priority })`, then `refreshTasks()`, then navigate to the task details page (e.g. `navigate(\`/tasks/${taskToEdit!.id}\`)`).

Keep validation (trimmed length, setError) and make `handleSubmit` an `async` function so you can `await createTask` / `await updateTask` and then `await refreshTasks()` before navigating. Example:

```ts
async function handleSubmit(e: React.FormEvent) {
  e.preventDefault();
  const trimmed = title.trim();
  if (trimmed.length === 0) {
    setError('Title is required.');
    return;
  }
  setError('');

  if (mode === 'create') {
    await createTask({ title: trimmed, priority });
    await refreshTasks();
    navigate('/tasks');
    return;
  }
  await updateTask(taskToEdit!.id, { title: trimmed, priority });
  await refreshTasks();
  navigate(`/tasks/${taskToEdit!.id}`);
}
```

Remove any code that built a `newTask` and called `onAdd(newTask)` or `onUpdate(...)`.

---

### When to move on

- The app loads tasks from **http://localhost:8000/api/tasks** when the backend is running.
- Add task, edit task, toggle completion, and delete task all call the API and the list stays in sync.
- After a refresh, the list still comes from the backend. Restarting the backend clears in-memory data (expected).

---

# Step 9 — End-to-end check and optional improvements

### Goal of this step

Confirm the full flow works and optionally add a small improvement (e.g. loading or error UI).

---

### Verification checklist

1. **Backend**

   - From `Exercise-3/backend`: `uvicorn main:app --reload`.
   - Backend uses layered architecture: routes in `routes/tasks.py` → `services/task_service.py` → `store.py`; `main.py` includes the router.
   - http://127.0.0.1:8000/docs shows the five task endpoints and you can call them.

2. **Frontend**

   - From the example project (e.g. `task-tracker-react-vite-ts`): `npm run dev`.
   - `.env` in the project root has `VITE_API_BASE_URL=http://localhost:8000`.

3. **Flow**

   - Open http://localhost:5173 (or your Vite URL).
   - List loads from the API (check Network tab: GET /api/tasks).
   - Add a task → POST /api/tasks, then list includes it.
   - Toggle completion → PATCH /api/tasks/{id}, list updates.
   - Edit task → PATCH, details/list update.
   - Delete → DELETE, task disappears.
   - Refresh page → list still comes from backend.

4. **CORS**
   - No CORS errors in the browser console when using the app.

---

## Summary

After completing this exercise you will have:

- A **FastAPI backend** under Exercise-3 with **layered architecture**:
  - **Routes (API layer)** in **routes/tasks.py** (dedicated file, like a .NET controller): handle HTTP, call the service, return JSON and status codes; **main.py** creates the app and includes the router.
  - **Service layer** in `services/task_service.py`: use cases (list, get, create, update, delete); calls the repository.
  - **Repository (data access layer)** in `store.py`: in-memory list and CRUD; replaceable later with a database.
- A **Pydantic** Task model aligned with your frontend Task type.
- **CORS** enabled so the React app can call the API.
- **Tailwind CSS 4** and **DaisyUI** in the frontend (Step 5): Vite plugin, `@import "tailwindcss"` and `@plugin "daisyui"` in the main CSS file; use utility and component classes in your components.
- The **example Task Tracker frontend** (or your Exercise 2 app) updated with: an **API client** (`src/api/tasksApi.ts`) and **per-page data fetching** — App only renders routes; **TasksListPage** fetches the list and refetches after toggle/delete; **TaskDetailsPage** fetches the single task by id and refetches or navigates after actions; **TaskFormPage** in edit mode fetches the task to edit, and create/update call the API then navigate. No shared context; each page is responsible for the data it needs.

When you add a real database later, you keep the same API and service layer and only replace the repository (e.g. `store.py` with a database-backed repository).

---

# Reporting guide — demonstrating the application with screenshots (PDF report)

Remember to include your github classroom repository link in your report

Use screenshots to show that your Task Tracker full-stack application is working correctly. The goal is to give a short, clear proof that both the backend and frontend are running and that the main flows work.

## What to include

1. **Backend running**

   - Screenshot of **http://127.0.0.1:8000/docs** showing the Swagger UI with the task endpoints (GET/POST/PATCH/DELETE).

2. **Frontend — task list**

   - Screenshot of the **task list page** (e.g. http://localhost:5173) with at least one task visible, loaded from the API.

3. **Frontend — main flows (pick at least 2–3)**

   - **Add task:** List view or form with a newly added task (or the form right before/after submit).
   - **Task details:** Detail page for one task showing title, priority, completed state.
   - **Edit task:** Edit form with existing data, or the list/details after an edit.
   - **Toggle completion:** List or details view with a task shown as completed (checkbox checked).
   - **Delete:** List view after a task has been deleted (the task no longer appears).

4. **Optional but recommended**
   - One screenshot with the browser **Network** tab open showing a successful API call (e.g. `GET /api/tasks` or `POST /api/tasks`) to confirm the frontend is talking to your backend.

## Tips

- **Keep it short.** A few clear screenshots are better than many. Aim for about 4–6 images that cover backend + list + 2–3 main flows.
- **Add short captions** (e.g. “Backend docs”, “Task list from API”, “Edit task”) so the reader knows what each screenshot shows.
- Add the screenshots in to your exercise report and return the report in PDF format. (no template is provided)
- Ensure both backend and frontend are running when you take the screenshots, and that `.env` points the frontend to your backend URL (e.g. `http://localhost:8000`).

