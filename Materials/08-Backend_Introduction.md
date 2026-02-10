# Backend Introduction: The Server Side of Full-Stack

So far you’ve built **frontend** experiences: HTML, CSS, JavaScript, and React run in the **browser**. They handle what the user sees and how they interact with the app. But where does the data come from? Who checks that a user is allowed to do something? Who talks to the database? That’s the **backend**.

This topic gives you a clear picture of what the backend is, why it exists, and how it fits into the full-stack model you’re learning. Later you’ll build it with **Python** and **FastAPI** and connect it to **PostgreSQL**.

---

## The Big Idea: Frontend and Backend Have Different Jobs

- **Frontend (client)**  
  Runs in the browser. Shows UI, handles clicks and forms, and **sends HTTP requests** when it needs data or needs to save something.

- **Backend (server)**  
  Runs on a server. **Receives HTTP requests**, runs business logic, talks to the database, and **sends HTTP responses** (often JSON) back to the frontend.

The frontend does **not** talk to the database directly. It talks only to the backend. The backend is the single place that:

- Decides what data a user may see or change
- Validates and sanitizes input
- Reads from and writes to the database
- Sends structured data (e.g. JSON) back to the client

---

## Why We Need a Backend

### 1. Security and trust

Everything in the browser can be inspected and modified by the user. You cannot trust the frontend to enforce rules like “only the owner can delete this” or “this field must be a number.” The backend runs on your server, so you can enforce those rules in code that the user cannot change.

### 2. Central place for business logic

Rules like “a task has a title and a due date” or “an exercise belongs to a category” live in one place: the backend. The frontend just displays and collects data; the backend decides what is valid and how it’s stored.

### 3. Database access

Databases (e.g. PostgreSQL) are not exposed directly to the internet. Only the backend connects to the database. The frontend never has database credentials or SQL; it only calls your API.

### 4. One backend, many clients

The same backend can serve a web app (React), a mobile app, or other services. They all use the same API and the same business logic.

---

## Request–Response: What the Backend Does

When the frontend needs data or wants to save something:

1. **Frontend** sends an **HTTP request** to a **URL** (e.g. `GET /api/tasks` or `POST /api/tasks`).
2. **Backend** receives the request:
   - Identifies the **path** and **method** (GET, POST, PUT, DELETE, etc.).
   - Optionally checks **authentication** (who is this user?).
   - Runs **business logic** (e.g. “fetch all tasks for this user” or “create a new task”).
   - Talks to the **database** if needed.
3. **Backend** sends an **HTTP response**:
   - A **status code** (200, 201, 400, 404, 500, etc.).
   - Often a **body** in **JSON** (e.g. a list of tasks or the created task).

The backend is stateless in the sense that each request is handled independently. Any “memory” of the user (e.g. who is logged in) is typically stored in tokens, sessions, or the database, not only in the server’s RAM.

---

## Layered Architecture: Organising Backend Code

Backend code is often organised into **layers**. Each layer has a clear responsibility and talks mainly to the layer above or below it. This keeps the code easier to understand, test, and change.

### Typical layers (top to bottom)

1. **API / Routes/Controllers layer**  
   Receives HTTP requests and returns HTTP responses. It:

   - Maps URLs and methods to handlers (e.g. `GET /api/tasks` → “list tasks”).
   - Parses and validates request data (body, query params).
   - Calls the layer below (services) to do the work.
   - Turns the result (or errors) into status codes and JSON.

2. **Service / Business logic layer**  
   Contains the actual rules of your application. It:

   - Implements use cases (e.g. “create a task”, “mark task as done”).
   - Enforces business rules (e.g. “a task must have a title”, “only the owner can delete”).
   - Coordinates calls to the data layer; it usually does **not** know about HTTP or JSON.

3. **Data access / Repository layer**  
   Talks to the database (or other data sources). It:
   - Runs queries (SQL or via an ORM).
   - Returns plain data (e.g. lists of tasks, one task) to the service layer.
   - Hides database details so the rest of the app is not tied to a specific DB or schema.

### Why use layers?

- **Single responsibility** — Each layer does one kind of job. Routes handle HTTP; services handle rules; the data layer handles persistence.
- **Easier testing** — You can test business logic by calling services directly, without sending HTTP requests. You can test routes by mocking the service layer.
- **Easier changes** — If you switch databases or change the API format, you touch mainly one layer.
- **Clear flow** — A request flows **down** (routes → services → data), and the result flows **back up** (data → services → routes → response).

### Request flow through the layers

For a request like `GET /api/tasks`:

1. **Routes:** “This is GET /api/tasks; call the task service to list tasks.”
2. **Service:** “Give me all tasks” (might filter by user, apply rules).
3. **Data layer:** Runs the query, returns rows.
4. **Service:** Returns the list to the route.
5. **Routes:** Serialises the list to JSON and sends a 200 response.

You will see this pattern when building the backend with FastAPI: route functions call service functions, and services use repositories or database sessions to load and save data.

---

## APIs: The Contract Between Frontend and Backend

The backend exposes an **API** (Application Programming Interface): a set of **endpoints** (URLs + methods) and a **contract** for what the client sends and what the server returns.

Example:

| Method | Path           | Meaning       | Request body     | Response                     |
| ------ | -------------- | ------------- | ---------------- | ---------------------------- |
| GET    | `/api/tasks`   | List tasks    | —                | `[{ id, title, done }, ...]` |
| GET    | `/api/tasks/3` | Get one task  | —                | `{ id, title, done }`        |
| POST   | `/api/tasks`   | Create a task | `{ title, due }` | `{ id, title, due, done }`   |
| PATCH  | `/api/tasks/3` | Update a task | `{ done: true }` | `{ id, title, done }`        |
| DELETE | `/api/tasks/3` | Delete a task | —                | 204 or 200                   |

The frontend is built to use this contract. If the backend changes the shape of the JSON or the status codes, the frontend may need to change too. That’s why we often document the API (e.g. with OpenAPI/Swagger) and use types (e.g. TypeScript on the frontend, Pydantic on the backend) to keep both sides in sync.

---

## JSON: The Common Data Format

Frontend and backend usually exchange data as **JSON** (JavaScript Object Notation). It’s text-based and easy for both JavaScript and Python to read and write.

Example response from `GET /api/tasks`:

```json
[
  { "id": 1, "title": "Finish exercises", "due": "2026-02-10", "done": false },
  {
    "id": 2,
    "title": "Read backend materials",
    "due": "2026-02-09",
    "done": true
  }
]
```

The backend:

- Builds such structures in Python (e.g. lists and dicts).
- Serializes them to JSON in the HTTP response.

The frontend:

- Receives the response and parses JSON (e.g. `response.json()`).
- Uses the data in React state and UI.

---

## What the Backend Is Responsible For (Summary)

| Responsibility           | Why it belongs on the backend                                                  |
| ------------------------ | ------------------------------------------------------------------------------ |
| **Validation**           | Input from the client must be checked; the client can be bypassed or modified. |
| **Authorization**        | “Is this user allowed to do this?” is decided on the server.                   |
| **Database access**      | Only the server has credentials and runs queries.                              |
| **Business rules**       | One place for “how tasks work,” “how exercises relate to categories,” etc.     |
| **Structured responses** | Same JSON shape and status codes for all clients (web, mobile, etc.).          |

---

## How This Fits Your Course Stack

In this course you will:

- Build the **frontend** with **React + TypeScript** (and Vite, Tailwind, etc.).
- Build the **backend** with **Python** and **FastAPI**.
- Use **PostgreSQL** as the database; the backend will connect to it, not the frontend.

Flow:

1. User interacts with the React app.
2. React calls your FastAPI backend (e.g. `fetch('http://localhost:8000/api/tasks')`).
3. FastAPI handles the request, optionally talks to PostgreSQL, and returns JSON.
4. React updates the UI with the response.

The next topic introduces **Python** — the language you’ll use to implement this backend.

---

## What You Should Understand After This Topic

By the end, you should be able to:

- Explain the difference between frontend and backend and why the frontend doesn’t talk to the database directly.
- Describe the request–response cycle between browser and backend.
- Explain what layered architecture is, name typical layers (API/routes, services, data access), and why layering helps (separation of concerns, testability, maintainability).
- Explain why validation, authorization, and business logic belong on the backend.
- Understand that the backend exposes an API (endpoints + JSON contract) that the frontend uses.
- See how Python (FastAPI) and PostgreSQL fit into this picture in your course.
