# Assignment: Iteration 4 — Authentication and Authorization

**Project:** Exercise/Workout Tracker  
**Iteration:** 4 of 4  
**Focus:** Add user authentication and authorization so that users must log in and can only access their own exercises, and containerize the application with Docker.

**Prerequisite:** Completed [ASSIGNMENT_ITERATION_3.md](./ASSIGNMENT_ITERATION_3.md) (PostgreSQL Database and Exercise Create/Update). You will extend the existing backend and frontend with user registration, login, and protected exercise routes.

**Reference:** Use **Exercise 5** (Task Tracker — Authentication, Authorization, and Docker) from the course materials as your implementation example. Exercise 5 provides detailed code and step-by-step instructions for a similar stack (FastAPI, React, JWT, PostgreSQL). Adapt its patterns to the Exercise Tracker domain.

---

## Introduction

In Iteration 3 you built a database-backed backend with create and update for exercises, and a frontend with forms. All data was accessible to anyone. In this iteration you will:

1. **Add user authentication** — Registration and login with JWT. Users register with email and password; login returns a JWT access token.
2. **Add a users table and `user_id` to exercises** — Each exercise belongs to a user. Scope all exercise operations by the authenticated user.
3. **Protect exercise routes** — Require a valid token to access exercise endpoints. Unauthenticated requests return 401.
4. **Enforce authorization** — Users can only list, view, create, update, and delete their own exercises. Accessing another user's exercise returns 404 or 403.
5. **Add frontend auth** — Login and register pages, auth context, protected routes, and an API client that sends the Bearer token with each request. Unauthenticated users are redirected to login.
6. **Containerize with Docker** — Backend and PostgreSQL run in Docker containers via Docker Compose. The full stack starts with `docker compose up`.

Categories remain **public** (no login required for `GET /api/categories`). Exercise endpoints become protected.

---

## Overall goal

1. **Backend authentication:** `POST /api/auth/register`, `POST /api/auth/login`. JWT with `sub` (user id) and expiry. Use `python-jose`, `bcrypt`, `python-multipart` (required for OAuth2 form login).
2. **Backend authorization:** Add `User` ORM model and `user_id` (nullable) to `Exercise`. Filter and scope all exercise repository, service, and route logic by `user_id`. Use `get_current_user` dependency to obtain the authenticated user from the JWT.
3. **Protected routes:** `GET /api/exercises`, `GET /api/exercises/{id}`, `POST /api/exercises`, `PUT /api/exercises/{id}` require a valid token. Return 401 if missing or invalid.
4. **Frontend auth:** Auth context (user, token, login, logout), Login and Register pages, protected route wrapper, API client that adds `Authorization: Bearer <token>` and redirects to login on 401.
5. **Docker:** Containerize backend and PostgreSQL with Docker Compose. The stack (db, api, optionally frontend) runs with `docker compose up`. See Exercise 5 Part B for guidance.

---

## What you need before starting

- **Iteration 3 solution** — Backend with PostgreSQL, SQLAlchemy 2, exercise create/update; frontend with list, detail, create, and edit pages.
- **Exercise 5** — Familiarize yourself with its auth flow (register, login, `get_current_user`, scoped repository, protected routes, AuthContext, `apiFetch`, ProtectedRoute).
- **Python 3.10+**, Node.js, PostgreSQL.
- **Docker** — Docker Desktop (Windows/Mac) or Docker Engine + Docker Compose (Linux).
- Course materials on authentication and JWT (e.g. Materials 11), and on Docker (e.g. Materials 13).

---

## Part A — Authentication and Authorization

### A.1 — Auth dependencies and User model

- Install `python-jose[cryptography]`, `bcrypt`, `python-multipart`.
- Add `User` (or `UserEntity`) ORM model: `id`, `email` (unique), `hashed_password`, `created_at`.
- Add `user_id` (nullable, FK to users) to the `Exercise` ORM model so existing exercises can migrate; new exercises will have `user_id` set.
- Add `SECRET_KEY` to `.env`. Create and apply an Alembic migration (or SQL script) for the new table and column.

### A.2 — Auth routes (register, login)

- Create `routes/auth.py` with:
  - `POST /auth/register` — Accept `{ email, password }`, hash password with bcrypt, insert user, return `{ id, email }`. Return 400 if email already exists.
  - `POST /auth/login` — Use `OAuth2PasswordRequestForm`; `username` = email. Verify password with bcrypt. Return JWT with `sub` = user id and expiry. Return 401 on invalid credentials.
- Implement `get_current_user(token, db)` — Decode JWT, fetch user by id, return user or raise 401.
- Register the auth router under `/api`.

### A.3 — Scope exercises by user

- **Repository:** Add `user_id` parameter to `get_all_exercises`, `get_exercise_by_id`, `create_exercise`, `update_exercise`. Filter by `user_id` in queries; set `user_id` on new exercises; return `None` for get/update if the exercise does not belong to the user.
- **Service:** Pass `user_id` from the route layer to all exercise repository calls.
- **Routes:** Inject `current_user: User = Depends(get_current_user)` into all exercise routes. Pass `current_user.id` to the service. Categories routes can remain public (no auth dependency).
- Ensure unauthenticated requests to exercise endpoints return 401.

### A.4 — Verify backend

- Use Swagger (`/docs`) or Scalar to test: register, login, copy token, authorize with Bearer token, then call `GET /api/exercises`, `POST /api/exercises`. Without a token, exercise endpoints should return 401.

### A.5 — Frontend: Auth context and pages

- Create `AuthContext` — Store `user`, `token`, `login`, `logout`, `isReady`. Persist token and user in `localStorage`; restore on load.
- Create **Login** and **Register** pages (e.g. `/login`, `/register`). Use your existing UI stack (e.g. Tailwind/DaisyUI). Register posts to `/api/auth/register`; Login posts form data to `/api/auth/login` and stores the token and user.
- Add routes for `/login` and `/register` (public, no auth required).

### A.6 — Frontend: Protected routes and API client

- Create `apiFetch` (or update your API client) — Add `Authorization: Bearer <token>` to all requests. On 401, clear auth and redirect to `/login`.
- Update all exercise API calls to use this client.
- Create `ProtectedRoute` — If not authenticated, redirect to `/login` (preserving `from` in state for post-login redirect).
- Wrap exercise list, detail, create, and edit routes with `ProtectedRoute`.
- Wrap the app with `AuthProvider`.
- Add a logout button (e.g. in a header or layout) that clears auth and redirects to login.

### A.7 — Verify end-to-end

- Register a user, log in. Create and edit exercises. Log out and log in again — data should persist. Register a second user and confirm each user sees only their own exercises.

---

## Part B — Docker (required)

- Follow Exercise 5 Part B to containerize the backend and PostgreSQL.
- Create `backend/Dockerfile` — Python base image, install dependencies, run migrations on startup, start uvicorn.
- Create `docker-compose.yml` in the project root with `db` (PostgreSQL) and `api` (backend) services. Use a volume for database persistence.
- Configure `DATABASE_URL` and `SECRET_KEY` via environment variables. The API must connect to the database using the `db` service hostname.
- Run migrations (e.g. `alembic upgrade head`) when the API container starts.
- Optionally add a frontend container (Vite build + nginx) to run the full stack in Docker.
- Ensure CORS allows the frontend origin (e.g. `http://localhost:5173`).
- Verify with `docker compose up -d --build`; the API and database should start successfully.

---

## Part C — Frontend design update (optional, graded)

You may update the frontend’s visual design and styling as you see fit. This part is **optional** — you can skip it and still earn full points for Parts A and B. However, if you complete it, it **counts toward your total points** and can improve your grade.

- **Scope:** Update styles, layout, colors, typography, spacing, or component appearance. You can refine existing pages (list, detail, create, edit, login, register) or add new UI elements (e.g. improved header, better forms, loading states).
- **Tools:** Use your existing stack (e.g. Tailwind CSS, DaisyUI) or introduce additional styling approaches if desired.
- **Guidelines:** Maintain functionality; the app must still meet all Part A and B requirements. Focus on clarity, consistency, and a cohesive look across the application.
- **Reporting:** If you complete Part C, describe your design changes and include before/after or updated screenshots in the report. This is required to receive points for Part C.

---

## Grading

Points are allocated as follows (or as specified by your course):

| Part | Description | Points |
|------|-------------|--------|
| **Part A** | Authentication and authorization (backend + frontend auth) | Majority of points |
| **Part B** | Docker setup (db + api containers, docker-compose) | Required for full base score |
| **Part C** | Frontend design update (optional) | Additional points if completed |

Part C is **optional**. You can earn full points for Parts A and B without completing it. If you complete Part C, you receive additional points toward your total. Document your design changes in the report (with screenshots) to receive Part C points.

---

## Summary

After completing this assignment you will have:

- **Authentication** — User registration and login; JWT-based sessions; `get_current_user` dependency.
- **Authorization** — Exercises scoped by `user_id`; users can only access their own exercises.
- **Protected backend** — Exercise endpoints require a valid token; 401 when unauthenticated.
- **Protected frontend** — Login and register pages; auth context; protected routes; API client with token; logout.
- **Docker** — Backend and database (and optionally frontend) run in containers; `docker compose up` starts the stack.
- **Frontend design (optional, graded)** — You may update styling and visual design; completing Part C earns additional points.

---

## Reporting

If your course requires a report for this iteration, document:

- Auth setup (User model, JWT flow, `get_current_user`, protected routes).
- How exercises are scoped by user (repository, service, routes).
- Frontend auth (AuthContext, login/register, protected routes, token in API client).
- Screenshots: login page, exercise list after login, API docs showing auth endpoints and a protected endpoint called with a token; Docker containers running.
- Docker: `docker-compose.yml` structure, how to run the stack with `docker compose up`, and data persistence in volumes.
- Frontend design (Part C): If you completed Part C, describe your design changes and include before/after or updated screenshots. Required to receive points for Part C.

---

*End of Assignment — Iteration 4*
