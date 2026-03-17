# Authentication and Authorization: Securing Your Fullstack Application

So far your Task Tracker API has been **open**: anyone who knows the URL can fetch tasks, add them, edit them, and delete them. For a personal or demo app that may be fine. For real applications you need to know **who** is making the request and **what** they are allowed to do.

This topic introduces **authentication** (who is the user?) and **authorization** (what may they do?). You'll learn how they fit into the request flow, common approaches (sessions, JWT), and how to implement auth on both the **backend** (FastAPI, JWT, password hashing) and the **frontend** (login form, token storage, protected routes). We include practical code examples you can adapt for your Task Tracker or other projects.

**Prerequisites:** Backend with FastAPI (Materials 08), database with PostgreSQL and SQLAlchemy (Materials 10). Understanding of HTTP, REST APIs, and the frontend–backend flow.

**The full picture:** A user logs in with email and password. The backend verifies the password (against a hash in the database), creates a **JWT**, and returns it. The frontend stores the token and sends it with every API request. The backend validates the token and uses it to identify the user. For each operation (e.g. list tasks), the backend filters data by that user — so Alice only sees her tasks, and Bob only sees his.

---

## Authentication vs Authorization

These terms are often used together but mean different things:

| Term               | Question it answers      | Example                                                                                                       |
| ------------------ | ------------------------ | ------------------------------------------------------------------------------------------------------------- |
| **Authentication** | _Who_ is this user?      | "This request comes from alice@example.com" — we have verified their identity (e.g. via password).            |
| **Authorization**  | _What_ may this user do? | "Alice is allowed to delete her own tasks but not others'" — we check permissions after we know who they are. |

**Order of operations:** You always authenticate first, then authorize. If you don't know who the user is, you can't decide what they're allowed to do.

---

## Why Authentication and Authorization Matter

- **Privacy** — Users expect their data to be private. Without auth, anyone could read or modify another user's tasks.
- **Accountability** — You need to know who created or changed something (audit trails, support).
- **Business rules** — Many features depend on identity: "show only my tasks," "only admins can delete users," "this comment was posted by X."

The **backend** is the only place that can enforce this. The frontend can hide buttons or redirect to a login page, but a determined user can send API requests directly (e.g. with curl or Postman). The backend must verify every protected request.

### Backend vs frontend responsibilities

| Layer | Responsibility |
|-------|----------------|
| **Backend** | Store users and password hashes; verify credentials; issue and validate JWTs; enforce auth on every protected route; filter data by user (authorization). |
| **Frontend** | Show login/register forms; send credentials; store and send the token; hide protected UI from unauthenticated users; redirect to login on 401. |

The frontend improves UX (no one sees the task list before logging in). The backend provides security (even if someone bypasses the frontend, the API rejects them).

---

## How Auth Fits in the Request Flow

When a user is logged in and the frontend calls a protected API:

```
Frontend (React)                    Backend (FastAPI)
     |                                    |
     |  GET /api/tasks                    |
     |  Authorization: Bearer <token>     |
     |  --------------------------------> |
     |                                    | 1. Extract token from header
     |                                    | 2. Validate token → identify user
     |                                    | 3. Load tasks for that user
     |                                    | 4. Return JSON
     |  <-------------------------------- |
     |  200 OK, [{ tasks }]               |
```

For an **unauthenticated** request to a protected endpoint:

```
     |  GET /api/tasks                    |
     |  (no token or invalid token)       |
     |  --------------------------------> |
     |                                    | 1. No valid token
     |                                    | 2. Return 401 Unauthorized
     |  <-------------------------------- |
     |  401 Unauthorized                  |
```

The frontend stores a **token** (or session id) after login and sends it with every request. The backend validates it and uses it to identify the user.

---

## Common Approaches: Sessions vs Tokens

### Session-based auth

- User logs in with username/password.
- Backend creates a **session** (stored in memory, Redis, or the database) and sends a **session ID** to the client (usually in a cookie).
- The client sends the cookie with each request.
- The backend looks up the session and gets the user.

**Pros:** Simple, server controls sessions (can invalidate immediately).  
**Cons:** Server must store sessions; scaling across multiple servers needs shared storage (e.g. Redis).

### Token-based auth (JWT)

- User logs in with username/password.
- Backend verifies credentials and issues a **JWT** (JSON Web Token) — a signed string that contains user info (e.g. user id, expiry).
- The client stores the token (e.g. in `localStorage` or memory) and sends it in the `Authorization: Bearer <token>` header.
- The backend verifies the signature and reads the user from the token; no database lookup needed for each request.

**Pros:** Stateless; no server-side session storage; works well with mobile apps and multiple backends.  
**Cons:** Hard to revoke before expiry (unless you maintain a blocklist); token size grows if you put lots of data in it.

### OAuth / third-party login

- User logs in via Google, GitHub, etc.
- Your backend receives an authorization code, exchanges it for tokens, and creates or links a user in your database.
- You still need to issue your own session or JWT for your API.

---

## Technologies Used in the Examples

The code examples use several libraries and patterns. Here is what each does and why we use it.

### Backend

| Technology | Purpose |
|------------|---------|
| **bcrypt** | Password-hashing library. Slow by design (makes brute-force attacks harder). We use it directly to hash and verify passwords. |
| **python-jose** | Library for creating and validating **JWT** (JSON Web Tokens). We use it to encode the user id and expiry into a signed token. |
| **OAuth2PasswordBearer** | FastAPI security scheme. Tells FastAPI to look for a token in the `Authorization: Bearer <token>` header. Used by `Depends(oauth2_scheme)` to extract the token automatically. |
| **OAuth2PasswordRequestForm** | FastAPI dependency that parses `application/x-www-form-urlencoded` form data with `username` and `password` fields. The OAuth2 spec uses "username" — we use it for email. |
| **Depends** | FastAPI dependency injection. A route that declares `Depends(get_current_user)` runs the dependency first; if it raises an exception (e.g. 401), the route handler never runs. |

**JWT structure:** A JWT has three parts (header.payload.signature), separated by dots. The payload (base64-encoded JSON) contains claims such as `sub` (subject, e.g. user id) and `exp` (expiry). The server signs it with a secret so clients cannot forge tokens. The backend can verify the signature and read the payload without a database lookup.

### Frontend

| Technology | Purpose |
|------------|---------|
| **FormData** | Browser API for building `multipart/form-data` or `application/x-www-form-urlencoded` request bodies. We use it to send `username` and `password` to the login endpoint, matching what `OAuth2PasswordRequestForm` expects. |
| **localStorage** | Browser storage that persists across tabs and page refreshes. We store the token and user so the user stays logged in after a refresh. Alternatives: `sessionStorage` (cleared when tab closes) or in-memory only (lost on refresh, more secure against XSS). |
| **React Context** | React feature for sharing state without prop drilling. The `AuthProvider` wraps the app and provides `user`, `token`, `login`, and `logout` to any component via `useAuth()`. See Materials 12 — State Management. |

---

## Passwords: Never Store Them in Plain Text

When users register or change passwords, you must **hash** them before storing in the database. A hash is a one-way function: you can't recover the original password from the hash.

**Why?**

- If the database is leaked, attackers get hashes, not passwords.
- Use a slow, purpose-built algorithm: **bcrypt**, **scrypt**, or **Argon2**. Fast hash functions (e.g. MD5, SHA256 alone) are too quick for attackers to brute-force.

**Install:** `pip install bcrypt`

```python
import bcrypt

# When storing a new password (registration or password change):
hashed = bcrypt.hashpw(plain_password.encode("utf-8"), bcrypt.gensalt()).decode("utf-8")  # Store hashed in DB, never plain_password

# When verifying a login:
is_valid = bcrypt.checkpw(plain_password.encode("utf-8"), hashed.encode("utf-8"))  # Returns True/False
```

Never log or return passwords. Never compare plain passwords in your database. Validate password strength (length, complexity) on registration.

---

## Backend: Implementing Authentication and Authorization

### 1. User model and database

Add a `users` table to store accounts. Each task will reference a `user_id` so you can filter by owner.

```python
# db_models.py — add UserEntity and user_id to TaskEntity
# Import ForeignKey from sqlalchemy for TaskEntity.user_id

class UserEntity(Base):
    """User account for authentication; stores email and hashed password."""
    __tablename__ = "users"

    id: Mapped[int] = mapped_column(primary_key=True, autoincrement=True)
    email: Mapped[str] = mapped_column(String(255), unique=True, nullable=False)
    hashed_password: Mapped[str] = mapped_column(String(255), nullable=False)  # Never store plain password
    created_at: Mapped[datetime] = mapped_column(DateTime, default=func.now())

# TaskEntity: add user_id so each task belongs to one user
class TaskEntity(Base):
    # ... existing columns (id, title, completed, priority, created_at, updated_at) ...
    user_id: Mapped[int] = mapped_column(ForeignKey("users.id"), nullable=False)  # Owner of the task
```

Create an Alembic migration to add the `users` table and `user_id` to `tasks`. Run `alembic upgrade head`.

### 2. Auth routes: register and login

**Install:** `pip install python-jose[cryptography] bcrypt`

```python
# routes/auth.py — registration and login endpoints
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
# OAuth2PasswordBearer: FastAPI extracts token from Authorization header for protected routes
oauth2_scheme = OAuth2PasswordBearer(tokenUrl="api/auth/login")

# JWT config: load SECRET_KEY from .env in production; never commit real secrets
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
    hashed = bcrypt.hashpw(body.password.encode("utf-8"), bcrypt.gensalt()).decode("utf-8")  # Hash before storing; never save plain password
    user = UserEntity(email=body.email, hashed_password=hashed)
    db.add(user)
    db.commit()
    return {"id": user.id, "email": user.email}  # Don't return hashed_password


@router.post("/login", response_model=Token)
def login(form: OAuth2PasswordRequestForm = Depends(), db: Session = Depends(get_db)):
    # OAuth2PasswordRequestForm expects "username" and "password"; we use username for email
    user = get_user_by_email(db, form.username)
    if not user or not bcrypt.checkpw(form.password.encode("utf-8"), user.hashed_password.encode("utf-8")):
        raise HTTPException(401, "Invalid email or password")
    # Build JWT payload: "sub" (subject) = user id; "exp" = expiry time
    expire = datetime.utcnow() + timedelta(minutes=ACCESS_TOKEN_EXPIRE_MINUTES)
    payload = {"sub": str(user.id), "exp": expire}
    token = jwt.encode(payload, SECRET_KEY, algorithm=ALGORITHM)
    return Token(access_token=token, user_id=user.id, email=user.email)
```

**Note:** `OAuth2PasswordRequestForm` expects `username` and `password` form fields. Use `username` for email when sending from the frontend.

### 3. JWT validation and get_current_user dependency

```python
# Dependency: runs before any route that uses Depends(get_current_user)
def get_current_user(token: str = Depends(oauth2_scheme), db: Session = Depends(get_db)) -> UserEntity:
    credentials_exception = HTTPException(401, "Invalid or expired token")
    try:
        # Decode and verify JWT; raises JWTError if signature invalid or token expired
        payload = jwt.decode(token, SECRET_KEY, algorithms=[ALGORITHM])
        user_id: str = payload.get("sub")  # "sub" contains the user id we stored at login
        if not user_id:
            raise credentials_exception
    except JWTError:
        raise credentials_exception
    user = db.get(UserEntity, int(user_id))
    if not user:
        raise credentials_exception  # User was deleted after token was issued
    return user
```

### 4. Protecting task routes

Inject `get_current_user` into every task route. Use `current_user.id` to filter and scope data:

```python
# Depends(get_current_user) runs first; if token invalid, FastAPI returns 401, route never runs
@router.get("/tasks", response_model=list[Task])
def list_tasks(
    current_user: UserEntity = Depends(get_current_user),
    db: Session = Depends(get_db),
):
    # Only return tasks belonging to this user
    return service.list_tasks_for_user(db, current_user.id)


@router.post("/tasks", response_model=Task, status_code=201)
def create_task(
    body: TaskCreate,
    current_user: UserEntity = Depends(get_current_user),
    db: Session = Depends(get_db),
):
    # Set task owner from the authenticated user
    return service.create_task(db, body, user_id=current_user.id)
```

Your repository must filter by `user_id`: `WHERE user_id = :user_id`. When creating a task, set `user_id` from the authenticated user.

### 5. Authorization: resource ownership

For update and delete, verify the task belongs to the current user:

```python
# Authorization: user is authenticated, but can they access this specific task?
@router.patch("/tasks/{task_id}")
def update_task(
    task_id: int,
    body: TaskUpdate,
    current_user: UserEntity = Depends(get_current_user),
    db: Session = Depends(get_db),
):
    task = service.get_task(db, task_id)
    if not task:
        raise HTTPException(404, "Task not found")
    # Ownership check: only the owner can update
    if task.user_id != current_user.id:
        raise HTTPException(403, "Forbidden: you can't edit this task")
    return service.update_task(db, task_id, body)
```

**401** = not logged in. **403** = logged in but not allowed to access this resource.

### 6. Public vs protected endpoints

| Endpoint | Auth required | Notes |
|----------|---------------|-------|
| `POST /api/auth/register` | No | Creates a new user |
| `POST /api/auth/login` | No | Returns JWT; use form data (`username`, `password`) |
| `GET /api/tasks` | Yes | Filter by `current_user.id` |
| `POST /api/tasks` | Yes | Set `user_id` from `current_user` |
| `GET /api/tasks/{id}` | Yes | Return 403 if `task.user_id != current_user.id` |
| `PATCH /api/tasks/{id}` | Yes | Same ownership check |
| `DELETE /api/tasks/{id}` | Yes | Same ownership check |

---

## Frontend: Implementing Authentication

### 1. Auth context: store user and token

Use React Context (see Materials 12 — State Management) so any component can access login state:

```tsx
// context/AuthContext.tsx — provides auth state to the whole app
import { createContext, useContext, useState, useEffect } from "react";

const API_BASE = import.meta.env.VITE_API_BASE_URL || "http://localhost:8000/api";

type User = { id: number; email: string };
type AuthContextType = {
  user: User | null;
  token: string | null;
  login: (email: string, password: string) => Promise<void>;
  logout: () => void;
  isReady: boolean;
};

const AuthContext = createContext<AuthContextType | null>(null);

const TOKEN_KEY = "auth_token";
const USER_KEY = "auth_user";

export function AuthProvider({ children }: { children: React.ReactNode }) {
  const [user, setUser] = useState<User | null>(null);
  const [token, setToken] = useState<string | null>(null);
  const [isReady, setIsReady] = useState(false);

  // On mount: restore user/token from localStorage (persists across refreshes)
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
    form.append("username", email);  // OAuth2PasswordRequestForm expects "username" (we use it for email)
    form.append("password", password);
    const res = await fetch(`${API_BASE}/auth/login`, {
      method: "POST",
      body: form,
    });
    if (!res.ok) throw new Error("Invalid email or password");
    const data = await res.json();
    setToken(data.access_token);
    setUser({ id: data.user_id, email: data.email });
    localStorage.setItem(TOKEN_KEY, data.access_token);
    localStorage.setItem(USER_KEY, JSON.stringify({ id: data.user_id, email: data.email }));
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
  if (!ctx) throw new Error("useAuth must be used within AuthProvider");
  return ctx;
}
```

**Storage trade-offs:** `localStorage` persists across tabs and refreshes but is vulnerable to XSS. For higher security, use memory only (token lost on refresh) or httpOnly cookies (backend must set them).

### 2. API client: attach token to every request

Create a wrapper that adds the `Authorization` header:

```tsx
// api/client.ts — use this instead of fetch; it adds the token and handles 401
const API_BASE = import.meta.env.VITE_API_BASE_URL || "http://localhost:8000/api";

async function getToken(): Promise<string | null> {
  return localStorage.getItem("auth_token");
}

export async function apiFetch(path: string, options: RequestInit = {}) {
  const token = await getToken();
  const headers: HeadersInit = { ...options.headers };
  // Add Authorization header so backend can identify the user
  if (token) {
    (headers as Record<string, string>)["Authorization"] = `Bearer ${token}`;
  }
  const res = await fetch(`${API_BASE}${path}`, { ...options, headers });
  // Token expired or invalid: clear auth state and redirect to login
  if (res.status === 401) {
    localStorage.removeItem("auth_token");
    localStorage.removeItem("auth_user");
    window.location.href = "/login";
    throw new Error("Unauthorized");
  }
  return res;
}
```

Use `apiFetch` instead of `fetch` for all API calls. On 401, clear storage and redirect to login.

### 3. Login page

```tsx
function LoginPage() {
  const { login } = useAuth();
  const navigate = useNavigate();
  const [email, setEmail] = useState("");
  const [password, setPassword] = useState("");
  const [error, setError] = useState("");

  const handleSubmit = async (e: React.FormEvent) => {
    e.preventDefault();
    setError("");
    try {
      await login(email, password);  // Calls API, stores token, updates context
      navigate("/");  // Redirect to app after successful login
    } catch {
      setError("Invalid email or password");
    }
  };

  return (
    <form onSubmit={handleSubmit}>
      <input type="email" value={email} onChange={(e) => setEmail(e.target.value)} required />
      <input type="password" value={password} onChange={(e) => setPassword(e.target.value)} required />
      {error && <p>{error}</p>}
      <button type="submit">Log in</button>
    </form>
  );
}
```

### 4. Protected routes (private routes)

**Private routes** are pages that require the user to be logged in. If not, redirect to the login page.

**Basic pattern — wrapper component**

```tsx
// Wraps routes that require login; redirects to /login if not authenticated
function ProtectedRoute({ children }: { children: React.ReactNode }) {
  const { user, isReady } = useAuth();
  const navigate = useNavigate();

  useEffect(() => {
    // isReady = we've checked localStorage; user = null means not logged in
    if (isReady && !user) {
      navigate("/login", { replace: true });  // replace: true = don't add to history
    }
  }, [isReady, user, navigate]);

  if (!isReady) return <p>Loading...</p>;  // Wait for auth check
  if (!user) return null;  // Will redirect; avoid flash of protected content
  return <>{children}</>;
}
```

**Alternative — using `Navigate`**

```tsx
import { Navigate, useLocation } from "react-router-dom";

function ProtectedRoute({ children }: { children: React.ReactNode }) {
  const { user, isReady } = useAuth();
  const location = useLocation();

  if (!isReady) return <p>Loading...</p>;
  if (!user) {
    // Pass location so we can redirect back after login
    return <Navigate to="/login" state={{ from: location }} replace />;
  }
  return <>{children}</>;
}
```

**Full router setup — public vs protected**

```tsx
// App.tsx — route structure
import { BrowserRouter, Routes, Route, Navigate } from "react-router-dom";
import { AuthProvider } from "./context/AuthContext";
import { ProtectedRoute } from "./components/ProtectedRoute";
import LoginPage from "./pages/LoginPage";
import TasksListPage from "./pages/TasksListPage";
import TaskDetailsPage from "./pages/TaskDetailsPage";

function App() {
  return (
    <AuthProvider>
      <BrowserRouter>
        <Routes>
          {/* Public routes — anyone can access */}
          <Route path="/login" element={<LoginPage />} />
          <Route path="/register" element={<RegisterPage />} />

          {/* Protected routes — require login */}
          <Route path="/" element={<ProtectedRoute><TasksListPage /></ProtectedRoute>} />
          <Route path="/tasks/:id" element={<ProtectedRoute><TaskDetailsPage /></ProtectedRoute>} />
          <Route path="/tasks/new" element={<ProtectedRoute><TaskFormPage /></ProtectedRoute>} />

          {/* Fallback: redirect unknown paths to home or 404 */}
          <Route path="*" element={<Navigate to="/" replace />} />
        </Routes>
      </BrowserRouter>
    </AuthProvider>
  );
}
```

**Redirect back after login**

When a user tries to visit a protected page (e.g. `/tasks/5`) but is not logged in, they go to `/login`. After logging in, redirect them back to the page they wanted:

```tsx
function LoginPage() {
  const { login } = useAuth();
  const navigate = useNavigate();
  const location = useLocation();

  const from = (location.state as { from?: { pathname: string } })?.from?.pathname || "/";

  const handleSubmit = async (e: React.FormEvent) => {
    e.preventDefault();
    // ... validation ...
    await login(email, password);
    navigate(from, { replace: true });  // Go back to the page they tried to visit
  };
  // ...
}
```

**Login-only route — redirect logged-in users**

If the user is already logged in and visits `/login`, redirect them to the app:

```tsx
function LoginPage() {
  const { user, isReady } = useAuth();
  const navigate = useNavigate();

  useEffect(() => {
    if (isReady && user) {
      navigate("/", { replace: true });  // Already logged in; no need to see login form
    }
  }, [isReady, user, navigate]);

  if (!isReady) return <p>Loading...</p>;
  if (user) return null;  // Will redirect
  // ... render login form ...
}
```

### 5. Logout

Add a logout button that calls `useAuth().logout()` and redirects to `/login`:

```tsx
function Header() {
  const { user, logout } = useAuth();
  const navigate = useNavigate();

  const handleLogout = () => {
    logout();
    navigate("/login", { replace: true });
  };

  return user ? <button onClick={handleLogout}>Log out</button> : null;
}
```

---

## Quick reference: backend vs frontend checklist

| Task | Backend | Frontend |
|------|---------|----------|
| Store users | `users` table, `hashed_password` | — |
| Register | `POST /auth/register` | Form → API |
| Login | `POST /auth/login`, verify password, return JWT | Form → API, store token |
| Validate token | `get_current_user` dependency | — |
| Send token | — | `Authorization: Bearer <token>` on every request |
| Filter by user | `WHERE user_id = current_user.id` | — |
| Handle 401 | Return 401 when token invalid | Clear token, redirect to login |
| Protect UI | — | `ProtectedRoute`, hide nav when logged out |

---

## Status Codes for Auth

| Code                 | Meaning                                                             | Typical use                               |
| -------------------- | ------------------------------------------------------------------- | ----------------------------------------- |
| **401 Unauthorized** | Not authenticated — no token, invalid token, or expired token.      | Missing or bad `Authorization` header.    |
| **403 Forbidden**    | Authenticated but not allowed — user is known but lacks permission. | User tries to delete another user's task. |

---

## Summary: Key Concepts

| Concept              | Meaning                                                                                  |
| -------------------- | ---------------------------------------------------------------------------------------- |
| **Authentication**   | Verifying _who_ the user is (e.g. via password or token).                                |
| **Authorization**    | Deciding _what_ the user may do (permissions, resource ownership).                       |
| **Session**          | Server-side storage of user state; client holds a session ID (often in a cookie).        |
| **JWT**              | A signed token containing user info; client sends it in `Authorization: Bearer <token>`. |
| **Password hashing** | Store only a hash (bcrypt, etc.); never plain text.                                      |
| **Protected route**  | Endpoint that requires a valid token/session before running.                             |
| **401**              | Not authenticated — invalid or missing token.                                            |
| **403**              | Authenticated but not authorised — e.g. accessing another user's resource.               |

---

## What You Should Understand After This Topic

By the end, you should be able to:

- Explain the difference between authentication and authorization.
- Describe how a token flows from login to protected API requests.
- Explain why passwords must be hashed and never stored in plain text.
- Understand the roles of sessions vs JWT (stateless tokens).
- Implement backend auth: User model, register/login routes, JWT creation and validation, `get_current_user` dependency.
- Protect routes with `Depends(get_current_user)` and filter data by `user_id`.
- Implement authorization checks for resource ownership (403 when user tries to access another's data).
- Implement frontend auth: Auth context, login form, API client that attaches the token and handles 401, protected routes.
