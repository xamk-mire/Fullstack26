# Exercise 5: Task Tracker — Authentication, Authorization, and Docker

## Important!

This exercise is a **follow-up to Exercise 4**. You will add **authentication and authorization** to the Task Tracker and wrap the backend and database in **Docker**.

Implement your solution inside your classroom exercises repository:

- Create a new folder **Exercise-5** in the repo.
- **Copy** your Exercise-4 backend and frontend into `Exercise-5/backend` and `Exercise-5/frontend`.
- You will **add** user registration/login, protect task routes, scope tasks by user, and **containerize** the backend and PostgreSQL with Docker.

**Prerequisites:** Completed Exercise 4 (PostgreSQL, SQLAlchemy, Task Tracker). Docker installed (Docker Desktop or Docker Engine). See **Materials 11 — Authentication and Authorization** and **Materials 13 — Docker** for background concepts.

---

## Overall goal

1. Add **user registration** and **login** (JWT-based).
2. Add a **users** table and **user_id** to tasks; scope all task operations by the authenticated user.
3. Protect task routes so only logged-in users can access them; enforce **authorization** (users can only access their own tasks).
4. Add **frontend** login page, auth context, protected routes, and API client that sends the token.
5. **Dockerize** the backend and PostgreSQL using Docker Compose so the whole stack runs with `docker compose up`.

By the end, the Task Tracker will require login, each user will see only their own tasks, and the backend + database will run in Docker containers.

---

## What you need before starting

- **Exercise 4 completed** — Backend with PostgreSQL, SQLAlchemy, Alembic; frontend connected to the API.
- **Frontend** — React app with **Tailwind CSS** and **DaisyUI** (see Materials 06 and 07). You will use them to style the login and register pages.
- **Docker** — Docker Desktop (Windows/Mac) or Docker Engine + Docker Compose (Linux).
- **Python 3.10+** and **Node.js** for local development and testing.

---

# Part A: Authentication and Authorization

---

# Step A1 — Install auth dependencies

From `Exercise-5/backend` (with venv activated):

```bash
pip install python-jose[cryptography] bcrypt python-multipart
```

**Note:** `python-multipart` is required for FastAPI to parse form data. The login endpoint uses `OAuth2PasswordRequestForm`, which expects `application/x-www-form-urlencoded`; without this package, login requests will fail with a parsing error.

---

# Step A2 — Add User model and user_id to Task

**A2.1** Add `UserEntity` to `db_models.py` and add `user_id` to `TaskEntity`. Add the `ForeignKey` import:

```python
# db_models.py — add these imports if missing
from sqlalchemy import String, DateTime, ForeignKey, func
```

Add the `UserEntity` class (before `TaskEntity`):

```python
class UserEntity(Base):
    """User account for authentication; stores email and hashed password."""
    __tablename__ = "users"

    id: Mapped[int] = mapped_column(primary_key=True, autoincrement=True)
    email: Mapped[str] = mapped_column(String(255), unique=True, nullable=False)
    hashed_password: Mapped[str] = mapped_column(String(255), nullable=False)
    created_at: Mapped[datetime] = mapped_column(DateTime, default=func.now())
```

Add `user_id` to `TaskEntity` (nullable to allow existing tasks from Exercise 4 to migrate without a user):

```python
# In TaskEntity, add after the existing columns:
user_id: Mapped[int | None] = mapped_column(ForeignKey("users.id"), nullable=True)
```

**A2.2** Add `SECRET_KEY` to `backend/.env`:

```env
SECRET_KEY=your-secret-key-change-in-production
DATABASE_URL=postgresql://postgres:postgres@localhost:5432/task_tracker
```

**A2.3** Create and apply the migration:

```bash
alembic revision --autogenerate -m "add users table and user_id to tasks"
alembic upgrade head
```

**Note on existing data:** With `nullable=True`, existing tasks from Exercise 4 will receive `user_id = NULL` when the migration runs. Those tasks will not appear in any user's list (the filter excludes them). New tasks created after login will have `user_id` set correctly.

---

# Step A3 — Add auth routes (register, login)

Create `backend/routes/auth.py`:

```python
import os
import bcrypt
from datetime import datetime, timedelta
from fastapi import APIRouter, Depends, HTTPException
from fastapi.security import OAuth2PasswordBearer, OAuth2PasswordRequestForm
from jose import JWTError, jwt
from pydantic import BaseModel
from sqlalchemy.orm import Session

from database import get_db
from db_models import UserEntity

router = APIRouter(prefix="/auth", tags=["auth"])
oauth2_scheme = OAuth2PasswordBearer(tokenUrl="api/auth/login")

SECRET_KEY = os.getenv("SECRET_KEY", "dev-secret-change-in-production")
ALGORITHM = "HS256"
ACCESS_TOKEN_EXPIRE_MINUTES = 60


class UserCreate(BaseModel):
    email: str
    password: str


class Token(BaseModel):
    access_token: str
    token_type: str = "bearer"
    user_id: int
    email: str


def get_user_by_email(db: Session, email: str) -> UserEntity | None:
    return db.query(UserEntity).filter(UserEntity.email == email).first()


@router.post("/register", status_code=201)
def register(body: UserCreate, db: Session = Depends(get_db)):
    if get_user_by_email(db, body.email):
        raise HTTPException(400, "Email already registered")
    hashed = bcrypt.hashpw(body.password.encode("utf-8"), bcrypt.gensalt()).decode("utf-8")
    user = UserEntity(email=body.email, hashed_password=hashed)
    db.add(user)
    db.commit()
    db.refresh(user)
    return {"id": user.id, "email": user.email}


@router.post("/login", response_model=Token)
def login(form: OAuth2PasswordRequestForm = Depends(), db: Session = Depends(get_db)):
    user = get_user_by_email(db, form.username)
    if not user or not bcrypt.checkpw(form.password.encode("utf-8"), user.hashed_password.encode("utf-8")):
        raise HTTPException(401, "Invalid email or password")
    expire = datetime.utcnow() + timedelta(minutes=ACCESS_TOKEN_EXPIRE_MINUTES)
    payload = {"sub": str(user.id), "exp": expire}
    token = jwt.encode(payload, SECRET_KEY, algorithm=ALGORITHM)
    return Token(access_token=token, user_id=user.id, email=user.email)
```

Add `get_current_user` to the same file (it will be used by task routes):

```python
def get_current_user(token: str = Depends(oauth2_scheme), db: Session = Depends(get_db)) -> UserEntity:
    credentials_exception = HTTPException(401, "Invalid or expired token")
    try:
        payload = jwt.decode(token, SECRET_KEY, algorithms=[ALGORITHM])
        user_id: str = payload.get("sub")
        if not user_id:
            raise credentials_exception
    except JWTError:
        raise credentials_exception
    user = db.get(UserEntity, int(user_id))
    if not user:
        raise credentials_exception
    return user
```

**A3.1** Include the auth router in `main.py`:

```python
from routes.auth import router as auth_router

app.include_router(auth_router, prefix="/api")
```

---

# Step A4 — Update Pydantic Task model, repository, service, and routes

**A4.1** Add `userId` to the Pydantic `Task` model in `models.py` (for ownership checks; optional because `user_id` can be null for migrated tasks):

```python
# In models.py Task class, add (with Optional from typing if needed):
userId: Optional[int] = None  # Owner of the task; used for authorization checks; null for legacy tasks
```

**A4.2** Update `repositories/task_repository.py` to filter and scope by `user_id`:

- `get_all_tasks(db, user_id)` — add `WHERE user_id = :user_id` to the select
- `get_task_by_id(db, task_id, user_id)` — add `user_id`; return the task only if `entity.user_id == user_id`, else `None`
- `create_task(db, body, user_id)` — add `user_id` parameter; set it on the entity (use `user_id: int = None` if you need to support nullable for legacy)
- `update_task(db, task_id, body, user_id)` — add `user_id`; load entity, return `None` if `entity.user_id != user_id`, else update and return the task
- `delete_task(db, task_id, user_id)` — add `user_id`; return `False` if task does not exist or `entity.user_id != user_id`, else delete and return `True`

Example implementation:

```python
def _entity_to_task(entity: TaskEntity) -> Task:
    return Task(
        id=entity.id,
        title=entity.title,
        completed=entity.completed,
        priority=Priority(entity.priority),
        createdAt=entity.created_at,
        updatedAt=entity.updated_at,
        userId=entity.user_id,
    )

def get_all_tasks(db: Session, user_id: int) -> list[Task]:
    stmt = select(TaskEntity).where(TaskEntity.user_id == user_id)
    entities = list(db.scalars(stmt))
    return [_entity_to_task(e) for e in entities]

def get_task_by_id(db: Session, task_id: int, user_id: int) -> Task | None:
    entity = db.get(TaskEntity, task_id)
    if not entity:
        return None
    if entity.user_id != user_id:
        return None  # Task does not belong to the user
    return _entity_to_task(entity)

def create_task(db: Session, body: TaskCreate, user_id: int = None) -> Task:
    entity = TaskEntity(
        title=body.title,
        completed=body.completed,
        priority=body.priority.value,
        user_id=user_id,
    )
    db.add(entity)
    db.commit()
    db.refresh(entity)
    return _entity_to_task(entity)

def update_task(db: Session, task_id: int, body: TaskUpdate, user_id: int) -> Task | None:
    entity = db.get(TaskEntity, task_id)
    if not entity:
        return None
    if entity.user_id != user_id:
        return None  # Task does not belong to the user
    updates = body.model_dump(exclude_unset=True)
    for key, value in updates.items():
        if key == "title":
            entity.title = value
        elif key == "completed":
            entity.completed = value
        elif key == "priority":
            entity.priority = value.value if hasattr(value, "value") else value
    db.commit()
    db.refresh(entity)
    return _entity_to_task(entity)

def delete_task(db: Session, task_id: int, user_id: int) -> bool:
    entity = db.get(TaskEntity, task_id)
    if not entity:
        return False
    if entity.user_id != user_id:
        return False  # Task does not belong to the user
    db.delete(entity)
    db.commit()
    return True
```

**A4.3** Update `services/task_service.py` to accept and pass `user_id` for all operations:

```python
def list_tasks(db: Session, user_id: int) -> list[Task]:
    return repo_get_all(db, user_id)

def get_task(db: Session, task_id: int, user_id: int) -> Task | None:
    return repo_get_by_id(db, task_id, user_id)

def create_task(db: Session, body: TaskCreate, user_id: int = None) -> Task:
    return repo_create(db, body, user_id)

def update_task(db: Session, task_id: int, body: TaskUpdate, user_id: int) -> Task | None:
    return repo_update(db, task_id, body, user_id)

def delete_task(db: Session, task_id: int, user_id: int) -> bool:
    return repo_delete(db, task_id, user_id)
```

**A4.4** Update `routes/tasks.py` to use `get_current_user` and enforce authorization:

```python
from fastapi import APIRouter, Depends, HTTPException
from sqlalchemy.orm import Session

from database import get_db
from db_models import UserEntity
from routes.auth import get_current_user
from models import Task, TaskCreate, TaskUpdate
from services.task_service import (
    list_tasks as service_list_tasks,
    get_task as service_get_task,
    create_task as service_create_task,
    update_task as service_update_task,
    delete_task as service_delete_task,
)

router = APIRouter(prefix="/tasks", tags=["tasks"])

def _check_ownership(task: Task, current_user: UserEntity):
    if task.userId is None or task.userId != current_user.id:
        raise HTTPException(403, "Forbidden: you can't access this task")

@router.get("", response_model=list[Task])
def list_tasks(
    current_user: UserEntity = Depends(get_current_user),
    db: Session = Depends(get_db),
):
    return service_list_tasks(db, current_user.id)

@router.get("/{task_id}", response_model=Task)
def get_task(
    task_id: int,
    current_user: UserEntity = Depends(get_current_user),
    db: Session = Depends(get_db),
):
    task = service_get_task(db, task_id, current_user.id)
    if not task:
        raise HTTPException(404, "Task not found")
    _check_ownership(task, current_user)
    return task

@router.post("", response_model=Task, status_code=201)
def post_task(
    body: TaskCreate,
    current_user: UserEntity = Depends(get_current_user),
    db: Session = Depends(get_db),
):
    return service_create_task(db, body, current_user.id)

@router.patch("/{task_id}", response_model=Task)
def patch_task(
    task_id: int,
    body: TaskUpdate,
    current_user: UserEntity = Depends(get_current_user),
    db: Session = Depends(get_db),
):
    task = service_get_task(db, task_id, current_user.id)
    if not task:
        raise HTTPException(404, "Task not found")
    _check_ownership(task, current_user)
    return service_update_task(db, task_id, body, current_user.id)

@router.delete("/{task_id}", status_code=204)
def remove_task(
    task_id: int,
    current_user: UserEntity = Depends(get_current_user),
    db: Session = Depends(get_db),
):
    task = service_get_task(db, task_id, current_user.id)
    if not task:
        raise HTTPException(404, "Task not found")
    _check_ownership(task, current_user)
    if not service_delete_task(db, task_id, current_user.id):
        raise HTTPException(404, "Task not found")
    return None
```

**A4.5** Ensure the task router is included in `main.py` (it should already be). The auth router and task router both use prefix `/api`, so the full paths are `/api/auth/...` and `/api/tasks/...`.

---

# Step A4.6 — Checkup: Test the backend with Scalar API docs

Before moving to the frontend, verify that the backend works correctly using **Scalar**, an interactive API documentation UI.

**A4.6.1** Install Scalar for FastAPI:

```bash
pip install scalar-fastapi
```

**A4.6.2** Add the Scalar route to `main.py`:

```python
from scalar_fastapi import get_scalar_api_reference

# Add this route (e.g. after the root route, before include_router):
@app.get("/scalar", include_in_schema=False)
async def scalar_html():
    return get_scalar_api_reference(
        openapi_url=app.openapi_url,
        title="Task Tracker API",
    )
```

**A4.6.3** Start the backend and open Scalar:

```bash
uvicorn main:app --reload
```

Open **http://localhost:8000/scalar** in your browser.

**A4.6.4** Test the auth flow and protected endpoints:

1. **Register** — In Scalar, find `POST /api/auth/register`. Click "Try it out", enter JSON (e.g. `{"email": "test@example.com", "password": "secret123"}`), execute. Expect 201.
2. **Login** — Find `POST /api/auth/login`. Click "Try it out". Use **x-www-form-urlencoded** (or form fields): `username` = `test@example.com`, `password` = `secret123`. Execute. Copy the `access_token` from the response.
3. **Authorize** — Click the "Authorize" or lock icon. For **OAuth2 (password)**, enter your email as username and password, or paste the token directly if Scalar offers a Bearer field. If there is a Bearer field, paste just the token (without "Bearer ").
4. **List tasks** — Call `GET /api/tasks`. Expect 200 with an empty array `[]` (or your tasks).
5. **Create task** — Call `POST /api/tasks` with body `{"title": "Test task", "completed": false, "priority": "medium"}`. Expect 201 with the created task.
6. **List tasks again** — Call `GET /api/tasks`. Expect 200 with a new "Test task" task in the response array

If all steps pass, the backend is working correctly. Proceed to the frontend.

---

# Step A5 — Frontend: Auth context and login page

**A5.1** Create `frontend/src/context/AuthContext.tsx`:

```tsx
import { createContext, useContext, useState, useEffect } from 'react';

const API_BASE =
  import.meta.env.VITE_API_BASE_URL || 'http://localhost:8000/api';

type User = { id: number; email: string };
type AuthContextType = {
  user: User | null;
  token: string | null;
  login: (email: string, password: string) => Promise<void>;
  logout: () => void;
  isReady: boolean;
};

const AuthContext = createContext<AuthContextType | null>(null);

const TOKEN_KEY = 'auth_token';
const USER_KEY = 'auth_user';

export function AuthProvider({ children }: { children: React.ReactNode }) {
  const [user, setUser] = useState<User | null>(null);
  const [token, setToken] = useState<string | null>(null);
  const [isReady, setIsReady] = useState(false);

  useEffect(() => {
    const savedToken = localStorage.getItem(TOKEN_KEY);
    const savedUser = localStorage.getItem(USER_KEY);
    if (savedToken && savedUser) {
      setToken(savedToken);
      setUser(JSON.parse(savedUser));
    }
    setIsReady(true);
  }, []);

  const login = async (email: string, password: string) => {
    const form = new FormData();
    form.append('username', email);
    form.append('password', password);
    const res = await fetch(`${API_BASE}/auth/login`, {
      method: 'POST',
      body: form,
    });
    if (!res.ok) throw new Error('Invalid email or password');
    const data = await res.json();
    setToken(data.access_token);
    setUser({ id: data.user_id, email: data.email });
    localStorage.setItem(TOKEN_KEY, data.access_token);
    localStorage.setItem(
      USER_KEY,
      JSON.stringify({ id: data.user_id, email: data.email }),
    );
  };

  const logout = () => {
    setToken(null);
    setUser(null);
    localStorage.removeItem(TOKEN_KEY);
    localStorage.removeItem(USER_KEY);
  };

  return (
    <AuthContext.Provider value={{ user, token, login, logout, isReady }}>
      {children}
    </AuthContext.Provider>
  );
}

export function useAuth() {
  const ctx = useContext(AuthContext);
  if (!ctx) throw new Error('useAuth must be used within AuthProvider');
  return ctx;
}
```

**A5.2** Create a Register page. Create `frontend/src/pages/RegisterPage.tsx`. **Style it with Tailwind CSS and DaisyUI** (see Materials 06 — Tailwind CSS and 07 — DaisyUI if your project does not yet have them). Use DaisyUI components such as `card`, `input`, `btn`, and `alert` for a consistent look:

```tsx
import { useState } from 'react';
import { useNavigate, Link } from 'react-router-dom';

const API_BASE =
  import.meta.env.VITE_API_BASE_URL || 'http://localhost:8000/api';

export default function RegisterPage() {
  const [email, setEmail] = useState('');
  const [password, setPassword] = useState('');
  const [error, setError] = useState('');
  const navigate = useNavigate();

  const handleSubmit = async (e: React.FormEvent) => {
    e.preventDefault();
    setError('');
    try {
      const res = await fetch(`${API_BASE}/auth/register`, {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ email, password }),
      });
      if (!res.ok) {
        const data = await res.json().catch(() => ({}));
        throw new Error(data.detail || 'Registration failed');
      }
      navigate('/login');
    } catch (err) {
      setError(err instanceof Error ? err.message : 'Registration failed');
    }
  };

  return (
    <div className="min-h-screen flex items-center justify-center bg-base-200 p-4">
      <div className="card w-full max-w-md bg-base-100 shadow-xl">
        <div className="card-body">
          <h1 className="card-title text-2xl">Register</h1>
          <form onSubmit={handleSubmit} className="space-y-4">
            <label className="form-control w-full">
              <span className="label-text">Email</span>
              <input
                type="email"
                placeholder="you@example.com"
                value={email}
                onChange={(e) => setEmail(e.target.value)}
                className="input input-bordered w-full"
                required
              />
            </label>
            <label className="form-control w-full">
              <span className="label-text">Password</span>
              <input
                type="password"
                placeholder="••••••••"
                value={password}
                onChange={(e) => setPassword(e.target.value)}
                className="input input-bordered w-full"
                required
              />
            </label>
            {error && (
              <div className="alert alert-error">
                <span>{error}</span>
              </div>
            )}
            <button type="submit" className="btn btn-primary w-full">
              Register
            </button>
          </form>
          <p className="text-center text-sm mt-2">
            <Link to="/login" className="link link-hover">
              Already have an account? Log in
            </Link>
          </p>
        </div>
      </div>
    </div>
  );
}
```

**A5.3** Create the Login page. Create `frontend/src/pages/LoginPage.tsx`. **Style it with Tailwind CSS and DaisyUI** (same approach as the Register page): use a centered card, form controls with labels, `input input-bordered`, `btn btn-primary`, and `alert alert-error` for errors.

```tsx
import { useState, useEffect } from 'react';
import { useNavigate, useLocation, Link } from 'react-router-dom';
import { useAuth } from '../context/AuthContext';

export default function LoginPage() {
  const { login, user, isReady } = useAuth();
  const navigate = useNavigate();
  const location = useLocation();
  const [email, setEmail] = useState('');
  const [password, setPassword] = useState('');
  const [error, setError] = useState('');

  const from =
    (location.state as { from?: { pathname: string } })?.from?.pathname || '/';

  useEffect(() => {
    if (isReady && user) {
      navigate(from, { replace: true });
    }
  }, [isReady, user, navigate, from]);

  const handleSubmit = async (e: React.FormEvent) => {
    e.preventDefault();
    setError('');
    try {
      await login(email, password);
      navigate(from, { replace: true });
    } catch {
      setError('Invalid email or password');
    }
  };

  if (!isReady || user) {
    return (
      <div className="min-h-screen flex items-center justify-center">
        <span className="loading loading-spinner loading-lg" />
      </div>
    );
  }

  return (
    <div className="min-h-screen flex items-center justify-center bg-base-200 p-4">
      <div className="card w-full max-w-md bg-base-100 shadow-xl">
        <div className="card-body">
          <h1 className="card-title text-2xl">Log in</h1>
          <form onSubmit={handleSubmit} className="space-y-4">
            <label className="form-control w-full">
              <span className="label-text">Email</span>
              <input
                type="email"
                placeholder="you@example.com"
                value={email}
                onChange={(e) => setEmail(e.target.value)}
                className="input input-bordered w-full"
                required
              />
            </label>
            <label className="form-control w-full">
              <span className="label-text">Password</span>
              <input
                type="password"
                placeholder="••••••••"
                value={password}
                onChange={(e) => setPassword(e.target.value)}
                className="input input-bordered w-full"
                required
              />
            </label>
            {error && (
              <div className="alert alert-error">
                <span>{error}</span>
              </div>
            )}
            <button type="submit" className="btn btn-primary w-full">
              Log in
            </button>
          </form>
          <p className="text-center text-sm mt-2">
            <Link to="/register" className="link link-hover">
              Don't have an account? Register
            </Link>
          </p>
        </div>
      </div>
    </div>
  );
}
```

---

# Step A6 — Frontend: API client, protected routes, and logout

**A6.1** Create or update your API client. Create `frontend/src/api/client.ts`:

```ts
const API_BASE =
  import.meta.env.VITE_API_BASE_URL || 'http://localhost:8000/api';

function getToken(): string | null {
  return localStorage.getItem('auth_token');
}

export async function apiFetch(path: string, options: RequestInit = {}) {
  const token = getToken();
  const headers: HeadersInit = { ...options.headers };
  if (token) {
    (headers as Record<string, string>)['Authorization'] = `Bearer ${token}`;
  }
  const res = await fetch(`${API_BASE}${path}`, { ...options, headers });
  if (res.status === 401) {
    localStorage.removeItem('auth_token');
    localStorage.removeItem('auth_user');
    window.location.href = '/login';
    throw new Error('Unauthorized');
  }
  return res;
}
```

**A6.2** Update all API calls in your frontend to use `apiFetch` instead of `fetch`. For example, if you fetch tasks:

```ts
// Before: fetch(`${API_BASE}/tasks`)
// After:  apiFetch("/tasks")
```

Use `/tasks` (not the full URL) since `apiFetch` prepends `API_BASE`.

**A6.3** Create `frontend/src/components/ProtectedRoute.tsx`:

```tsx
import { Navigate, useLocation } from 'react-router-dom';
import { useAuth } from '../context/AuthContext';

export default function ProtectedRoute({
  children,
}: {
  children: React.ReactNode;
}) {
  const { user, isReady } = useAuth();
  const location = useLocation();

  if (!isReady) return <p>Loading...</p>;
  if (!user) {
    return <Navigate to="/login" state={{ from: location }} replace />;
  }
  return <>{children}</>;
}
```

**A6.4** Wrap `App.tsx` (or your root) with `AuthProvider` and configure routes:

```tsx
import { BrowserRouter, Routes, Route, Navigate } from 'react-router-dom';
import { AuthProvider } from './context/AuthContext';
import ProtectedRoute from './components/ProtectedRoute';
import LoginPage from './pages/LoginPage';
import RegisterPage from './pages/RegisterPage';
import TasksListPage from './pages/TasksListPage'; // your task list page
import TaskDetailsPage from './pages/TaskDetailsPage'; // your task details page
// Add other pages as needed

function App() {
  return (
    <AuthProvider>
      <BrowserRouter>
        <Routes>
          <Route path="/login" element={<LoginPage />} />
          <Route path="/register" element={<RegisterPage />} />

          <Route
            path="/"
            element={
              <ProtectedRoute>
                <TasksListPage />
              </ProtectedRoute>
            }
          />
          <Route
            path="/tasks/:id"
            element={
              <ProtectedRoute>
                <TaskDetailsPage />
              </ProtectedRoute>
            }
          />
          {/* Wrap any other protected pages similarly */}

          <Route path="*" element={<Navigate to="/" replace />} />
        </Routes>
      </BrowserRouter>
    </AuthProvider>
  );
}
```

**A6.5** Add a Logout button. In your layout or header (e.g. a navbar or app shell that wraps protected content):

```tsx
import { useNavigate } from 'react-router-dom';
import { useAuth } from '../context/AuthContext';

export function Header() {
  const { user, logout } = useAuth();
  const navigate = useNavigate();

  const handleLogout = () => {
    logout();
    navigate('/login', { replace: true });
  };

  return user ? (
    <header>
      <span>Logged in as {user.email}</span>
      <button onClick={handleLogout}>Log out</button>
    </header>
  ) : null;
}
```

Include `Header` in your protected layout so it appears on task pages.

**A6.6** Update the Task type in your frontend to include `userId` if you use it (optional):

```ts
export interface Task {
  id: number;
  title: string;
  completed: boolean;
  priority: Priority;
  createdAt: string;
  updatedAt?: string;
  userId?: number; // optional; backend may include it
}
```

---

# Step A7 — Run and test (without Docker)

1. Start the backend: `uvicorn main:app --reload` from `Exercise-5/backend`
2. Start the frontend: `npm run dev` from `Exercise-5/frontend`
3. Ensure `frontend/.env` has `VITE_API_BASE_URL=http://localhost:8000`
4. Open the app: register a user, log in, add tasks. Try logging out and logging in again. Register a second user and confirm each user sees only their own tasks.

---

# Part B: Docker

---

# Step B1 — Create requirements.txt

Ensure `backend/requirements.txt` exists with all dependencies:

```
fastapi
uvicorn[standard]
sqlalchemy
psycopg2-binary
python-dotenv
alembic
python-jose[cryptography]
bcrypt
python-multipart
scalar-fastapi
```

If you use `pip freeze`, filter to include only these (and their dependencies) to avoid bloat. Or create the file manually as above.

---

# Step B2 — Add a Dockerfile for the backend

Create `backend/Dockerfile`:

```dockerfile
FROM python:3.12-slim

WORKDIR /app

COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

COPY . .

EXPOSE 8000

CMD alembic upgrade head && uvicorn main:app --host 0.0.0.0 --port 8000
```

**B2.1** (Optional) Create `backend/.dockerignore` to exclude unnecessary files:

```
__pycache__
*.pyc
.env
venv
.git
*.md
```

---

# Step B2.5 — Add a Dockerfile and nginx config for the frontend

The frontend is built with Vite and then served by nginx so it can run in a container. The browser will call the API at `http://localhost:8000`; we pass that URL at build time.

**B2.5.1** Create `frontend/nginx.conf` (for SPA routing — all routes serve `index.html`):

```nginx
server {
    listen 80;
    root /usr/share/nginx/html;
    index index.html;

    location / {
        try_files $uri $uri/ /index.html;
    }
}
```

**B2.5.2** Create `frontend/Dockerfile`:

```dockerfile
# Stage 1: Build the Vite app
FROM node:20-alpine AS build
WORKDIR /app

COPY package*.json ./
RUN npm ci

ARG VITE_API_BASE_URL=http://localhost:8000
ENV VITE_API_BASE_URL=$VITE_API_BASE_URL

COPY . .
RUN npm run build

# Stage 2: Serve with nginx
FROM nginx:alpine
COPY --from=build /app/dist /usr/share/nginx/html
COPY nginx.conf /etc/nginx/conf.d/default.conf
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

**Note:** `VITE_API_BASE_URL` is baked into the build so the browser knows where to send API requests. In docker-compose we pass it as a build arg.

**B2.5.3** (Optional) Create `frontend/.dockerignore` to speed up builds:

```
node_modules
dist
.env
.env.local
.git
*.md
```

---

# Step B3 — Create docker-compose.yml

Create `docker-compose.yml` in the **Exercise-5** root (same level as `backend/` and `frontend/`):

```yaml
services:
  db:
    image: postgres:16
    environment:
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: postgres
      POSTGRES_DB: task_tracker
    ports:
      - '5432:5432'
    volumes:
      - task_tracker_data:/var/lib/postgresql/data

  api:
    build: ./backend
    ports:
      - '8000:8000'
    environment:
      DATABASE_URL: postgresql://postgres:postgres@db:5432/task_tracker
      SECRET_KEY: ${SECRET_KEY:-dev-secret-change-in-production}
    depends_on:
      - db

  frontend:
    build:
      context: ./frontend
      dockerfile: Dockerfile
      args:
        VITE_API_BASE_URL: http://localhost:8000
    ports:
      - '5173:80'
    depends_on:
      - api

volumes:
  task_tracker_data:
```

**Important:**
- `DATABASE_URL` uses `db` as the host. Inside Docker Compose, services reach each other by service name.
- The **frontend** is built with `VITE_API_BASE_URL=http://localhost:8000` so the browser (running on your machine) can call the API. The frontend app is served at **http://localhost:5173** (port 5173 on host maps to port 80 in the container).
- Ensure CORS in `main.py` allows `http://localhost:5173` (and `http://127.0.0.1:5173`) so the frontend can call the API.

---

# Step B4 — Run and test with Docker

**B4.1** From the **Exercise-5** root, build and start all services (db, api, frontend):

```bash
docker compose up -d --build
```

The first time may take a few minutes while the frontend image is built. Check that all three containers are running:

```bash
docker compose ps
```

**B4.2** Open the app in your browser:

- **Frontend:** http://localhost:5173 (served by the frontend container)
- **API:** http://localhost:8000 (Swagger/Scalar at http://localhost:8000/docs or http://localhost:8000/scalar)

**B4.3** Test the full flow: register, log in, add tasks. Restart containers to verify persistence:

```bash
docker compose down
docker compose up -d
```

Data should persist (stored in the `task_tracker_data` volume).

**Optional — run frontend on the host for development:** If you prefer to run the frontend with `npm run dev` for hot reload, comment out or remove the `frontend` service from `docker-compose.yml` and run `npm run dev` from `frontend/`. Ensure `frontend/.env` has `VITE_API_BASE_URL=http://localhost:8000`.

**Troubleshooting:**

- If the API fails to start, check logs: `docker compose logs api`
- If migrations fail, ensure `alembic.ini` and `alembic/` are in the backend folder and that `env.py` uses `DATABASE_URL` from the environment.
- If the frontend cannot reach the API, ensure CORS in `main.py` allows `http://localhost:5173` and `http://127.0.0.1:5173`.
- If the frontend build fails, ensure the frontend app builds locally with `npm run build` and that `VITE_API_BASE_URL` is used in your API client (e.g. `import.meta.env.VITE_API_BASE_URL`).

---

## Verification checklist

1. **Auth** — Register and login work; JWT is returned; protected endpoints return 401 without a token.
2. **Authorization** — Users see only their own tasks; 403 when accessing another user's task (e.g. via direct API call with a valid token but wrong task id).
3. **Frontend** — Login and register pages work; protected routes redirect to login when not authenticated; API client sends token; 401 clears auth and redirects; logout works.
4. **Docker** — `docker compose up -d` starts db, api, and frontend; frontend at http://localhost:5173, API at http://localhost:8000; migrations run on startup; data persists in volume after restart.

---

## Summary

After completing this exercise you will have:

- **Authentication** — User registration, login, JWT; `get_current_user` dependency.
- **Authorization** — Tasks scoped by `user_id`; ownership check on get/update/delete (403).
- **Frontend auth** — Auth context, login/register pages, protected routes, API client with token, logout.
- **Docker** — Backend, PostgreSQL, and frontend in containers; `docker compose up` runs the full stack (db, api, frontend); frontend built with Vite and served by nginx.

---

# Reporting guide — demonstrating the application with screenshots (PDF report)

Remember to include your GitHub Classroom repository link in your report.

Use screenshots to show that your Task Tracker has **authentication**, **authorization**, and **Docker** working.

## What to include

1. **Docker** — Screenshot of `docker compose ps` (or `docker compose up`) showing the `db`, `api`, and `frontend` containers running.
2. **Login** — Screenshot of the login page or a successful login flow.
3. **Protected task list** — Screenshot of the task list after logging in (tasks loaded from API).
4. **API with auth** — Screenshot of Swagger (http://localhost:8000/docs) showing the auth endpoints and/or a protected endpoint called with a valid token (e.g. "Authorize" with token, then GET /api/tasks).
5. **Optional** — Screenshot of the `docker-compose.yml` or project structure.

## Tips

- Keep it short; 4–5 clear screenshots are sufficient.
- Add captions (e.g. "Docker containers running", "Login page", "Task list after login").
- Return the report in PDF format.
