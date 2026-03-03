# PostgreSQL and SQLAlchemy 2: Database Access in the Python Backend

So far you’ve built a FastAPI backend that stores tasks in **memory**. When the server restarts, the data is gone. For real applications you need **persistent storage** — a database. In this course we use **PostgreSQL** and **SQLAlchemy 2** as the ORM to connect your Python backend to the database.

This topic introduces PostgreSQL, SQLAlchemy 2, and how they fit into your layered backend. You’ll see how to define models, run queries, and integrate the database with FastAPI. We also cover **migrations** and **Alembic** — the standard way to evolve your schema over time instead of recreating tables from scratch. **Exercise 4** walks through updating your Task Tracker (Exercise 3) from in-memory storage to PostgreSQL.

---

## What Is PostgreSQL?

**PostgreSQL** (often shortened to "Postgres") is a **relational database management system** (RDBMS). It stores data in **tables** made up of rows and columns, with relationships between tables defined by foreign keys. You interact with it using **SQL** (Structured Query Language) — the standard language for querying and modifying relational data.

PostgreSQL runs as a **separate server process**. Your backend does not embed the database; instead, it connects to PostgreSQL over the network (or locally via a socket). This separation means you can scale the database independently, run it on a different machine, or swap it for another database without changing your application logic — as long as you use an abstraction layer like an ORM.

**Why PostgreSQL?**

- **Reliable** — ACID transactions ensure data consistency even when many operations run concurrently. If something fails mid-transaction, changes are rolled back.
- **Powerful** — Beyond traditional tables and SQL, PostgreSQL supports JSON/JSONB, full-text search, geospatial data, and many extensions. It handles complex queries and large datasets well.
- **Widely used** — A common choice for production web apps, from startups to large enterprises. It has a strong ecosystem and plenty of hosting options.
- **Open source** — Free to use, self-host, and extend. No licensing fees.

You can install PostgreSQL locally, run it in Docker, or use a managed cloud service (e.g. Supabase, Neon, Railway, AWS RDS). The backend connects via a **connection string**:

```
# Format: postgresql://user:password@host:port/database_name
postgresql://user:password@localhost:5432/database_name
```

---

## What Is an ORM? Why SQLAlchemy?

[SQLAlchemy docs](https://www.sqlalchemy.org/)

You _could_ write raw SQL in Python and send it to PostgreSQL. But that approach has drawbacks: string concatenation for dynamic queries opens the door to **SQL injection**, manually mapping rows to Python objects is tedious, and switching databases requires rewriting SQL dialects. An **ORM** (Object-Relational Mapper) addresses these issues by letting you work with **Python classes** instead of SQL strings. You define models that map to tables; the ORM generates SQL, parameterizes values safely, and maps query results back to objects.

The name captures the idea: **object-relational mapping** bridges the gap between **objects** (Python classes, with attributes and methods) and **relational data** (tables, rows, columns). Instead of writing `SELECT * FROM tasks WHERE id = ?` and then manually building a `Task` instance from the row, you write something like `session.get(Task, id)` and get a typed object back. The ORM handles the translation both ways — Python to SQL for queries and mutations, SQL results back to Python objects.

**Why SQLAlchemy?**

**SQLAlchemy** is the dominant Python ORM. It has been maintained since 2005 and is used across countless production applications — from small APIs to large systems. Unlike framework-specific ORMs (such as Django's), SQLAlchemy is **database-agnostic**: it works with PostgreSQL, MySQL, SQLite, and others through pluggable drivers. The same model and query code can target different databases with minimal changes. It also integrates well with FastAPI, Flask, and other Python web frameworks.

**SQLAlchemy 2** (2.0+, released in 2023) introduced a cleaner, more explicit API that aligns with modern Python practices. The old 1.x API is deprecated; new projects should use the 2.0 style:

- **Typed models** — `Mapped` and `mapped_column` give you proper type hints and better IDE support. Your models are easier to understand and refactor.
- **Explicit `select()`** — The new API uses `select()` constructs instead of the older `query` API, making it clearer what SQL is being generated. Queries are built step by step rather than through a chain of methods.
- **Context managers and sessions** — Clean patterns for opening and closing database connections. Sessions scope work to a unit (e.g. one HTTP request) and ensure resources are released.
- **Async support** — For high-concurrency web apps, SQLAlchemy 2 supports `AsyncSession` and async engines. You can choose sync or async depending on your needs.

SQLAlchemy does not hide SQL. When you need full control — a complex join, a database-specific feature, or a performance-critical query — you can drop to raw SQL. For most CRUD operations and common queries, the ORM provides a clean, maintainable Python interface that keeps your code readable and safe.

---

## How Databases Are Handled in Fullstack Applications

In a typical fullstack setup, the **database is never accessed directly by the frontend**. The flow is always:

```
Browser (React)  →  HTTP (REST API)  →  Backend (FastAPI)  →  Database (PostgreSQL)
```

**Why the frontend never talks to the database directly**

- **Security** — Database credentials must stay on the server. Exposing a database connection string or allowing direct DB access from the browser would put your data at risk.
- **Architecture** — Browsers cannot run PostgreSQL drivers. The frontend speaks HTTP/HTTPS and JSON; only the backend has the drivers and connection logic to talk to the database.
- **Control** — The backend enforces validation, authorization, and business rules. Users should only see and modify data through your API, not by sending arbitrary SQL.

**Request lifecycle**

When a user loads a task list in the browser:

1. The frontend sends `GET /api/tasks` to the backend.
2. The **route** receives the request and calls the **service**.
3. The **service** calls the **repository** (e.g. `get_all_tasks(db)`).
4. The **repository** uses a SQLAlchemy session to run a query against PostgreSQL.
5. The result is returned as domain objects → Pydantic models → JSON.
6. The frontend receives the JSON and renders the UI.

The database connection is created (or reused from a pool) when the request reaches the backend and is closed when the request completes. This keeps connections bounded and avoids leaking resources.

**Environment-specific configuration**

In development you might use a local PostgreSQL instance; in production, a managed cloud database. The backend reads the database URL from an environment variable (e.g. `DATABASE_URL`) so you can change it without changing code. Never commit credentials — use `.env` files (excluded from version control) or a secrets manager.

---

## The Big Picture: Where the Database Fits

In **layered architecture** (see Materials 08 — Backend Introduction), the database belongs in the **data access / repository layer**:

1. **Routes** — Receive HTTP, call services.
2. **Services** — Business logic, call repositories.
3. **Repository** — Talks to the database via SQLAlchemy; returns domain objects or lists.

Right now your repository might use an in-memory list. When you add PostgreSQL, you replace that with a repository that uses SQLAlchemy sessions. The service and routes stay the same — only the repository implementation changes.

---

## Migrations and Alembic

[Alembic docs](https://alembic.sqlalchemy.org/en/latest/index.html)

As your app grows, the database schema changes: you add columns, rename tables, create indexes. Without a structured approach, you might drop and recreate tables — which wipes all data — or hand-edit the database in production, which is error-prone and hard to reproduce. **Migrations** solve this by providing versioned scripts that describe schema changes in a repeatable, traceable way. Each migration is a small step (e.g. "add column X to table Y"); you apply them in order so dev, staging, and production stay in sync.

Migrations also help when working in a team: everyone applies the same sequence of changes, and the schema history lives in version control alongside your code. If a colleague adds a migration, you run `alembic upgrade head` and your local database matches theirs.

**What is Alembic?**

**Alembic** is the standard migration tool for SQLAlchemy. It was created by the same author (Mike Bayer) and is designed to work seamlessly with SQLAlchemy models. Unlike `Base.metadata.create_all()`, which only creates tables and cannot evolve an existing schema, Alembic tracks changes over time and applies them incrementally.

**How Alembic works**

- **Migration scripts** — Each migration is a Python file in `alembic/versions/` with an `upgrade()` function (apply the change) and a `downgrade()` function (revert it). You can generate scripts automatically from model changes with `alembic revision --autogenerate`, or write them by hand for complex or data migrations.
- **Version tracking** — Alembic stores which migrations have been applied in a special `alembic_version` table in your database. When you run `alembic upgrade head`, it applies only the migrations that haven't run yet. Each environment (dev, staging, production) maintains its own version; they all follow the same migration history.
- **Upgrade and downgrade** — `alembic upgrade head` applies all pending migrations. `alembic downgrade -1` reverts the last one. Downgrades are useful when you need to roll back a bad deployment or when developing and iterating on a migration.
- **Environment-agnostic** — Migrations are just Python scripts that use your `DATABASE_URL`. The same migration files work for every environment; only the connection string changes (via `.env` or environment variables).

---

## SQLAlchemy 2 Basics

### Install dependencies

```bash
# Install ORM, PostgreSQL driver, migration tool, and env loader.
pip install sqlalchemy psycopg2-binary alembic python-dotenv
```

- **sqlalchemy** — ORM and engine
- **psycopg2-binary** — PostgreSQL driver (sync). For async, use’d use `asyncpg` and `postgresql+asyncpg://` in the connection string.
- **alembic** — Database migrations
- **python-dotenv** — Load `DATABASE_URL` from `.env`

---

### 1. Create the engine

The **engine** manages connections to the database. Create it once at startup:

```python
from sqlalchemy import create_engine

# Connection string: user:password@host:port/database_name
DATABASE_URL = "postgresql://user:password@localhost:5432/task_tracker"

# Engine manages the connection pool; create once at app startup.
# echo=True logs all SQL to the console (useful for learning and debugging).
engine = create_engine(DATABASE_URL, echo=True)
```

Use an environment variable for `DATABASE_URL` in real projects so you don’t commit credentials.

---

### 2. Declare models with `DeclarativeBase`

Models are Python classes that map to database tables. SQLAlchemy 2 uses `**DeclarativeBase**` and typed columns with `**Mapped**` and `**mapped_column**`:

```python
from datetime import datetime
from typing import Optional
from sqlalchemy import String, DateTime, func
from sqlalchemy.orm import DeclarativeBase, Mapped, mapped_column


# Base class for all models; provides the mapping machinery.
class Base(DeclarativeBase):
    pass


# Task model maps to the "tasks" table in the database.
class Task(Base):
    __tablename__ = "tasks"

    # Primary key; database auto-generates on insert.
    id: Mapped[int] = mapped_column(primary_key=True, autoincrement=True)
    # Title column; max 255 chars, required (not null).
    title: Mapped[str] = mapped_column(String(255), nullable=False)
    # Completion flag; defaults to False for new tasks.
    completed: Mapped[bool] = mapped_column(default=False)
    # Priority string; defaults to "medium".
    priority: Mapped[str] = mapped_column(String(20), default="medium")
    # Set by the database on insert via func.now().
    created_at: Mapped[datetime] = mapped_column(DateTime, default=func.now())
    # Set by the database on update; None until first update.
    updated_at: Mapped[Optional[datetime]] = mapped_column(DateTime, default=None, onupdate=func.now())
```

- `**__tablename__**` — Table name in the database.
- `**Mapped[type]**` — Type hint for the attribute.
- `**mapped_column()**` — Column definition; `primary_key`, `nullable`, `default`, etc.
- `**Optional[datetime]**` — Allows `None` (nullable column); used for `updated_at`.

---

### 3. Create tables with Alembic

Instead of `Base.metadata.create_all()`, use **Alembic** so you can evolve the schema over time without losing data.

**Initialize Alembic** (run once):

```bash
# Creates alembic.ini, alembic/ directory, and alembic/versions/ for migration scripts.
alembic init alembic
```

**Configure `alembic/env.py`** — Add your project to `sys.path`, load `DATABASE_URL` from `.env`, and set `target_metadata = Base.metadata`. See **Exercise 4** for a full `env.py` example.

**Generate and apply the initial migration:**

```bash
# Compare models to DB, generate a migration script in alembic/versions/.
alembic revision --autogenerate -m "create tasks table"
# Apply all pending migrations to the database.
alembic upgrade head
```

Alembic compares your models to the database and generates a migration script. `upgrade head` applies all pending migrations. For future changes: edit models → `alembic revision --autogenerate -m "description"` → `alembic upgrade head`.

---

### 4. Session and CRUD

A **Session** represents a conversation with the database. Use it as a context manager so it is closed properly. The session tracks changes and emits SQL on `commit()`.

**Select all:**

```python
from sqlalchemy import select
from sqlalchemy.orm import Session

with Session(engine) as session:
    # Build a SELECT * FROM tasks query.
    stmt = select(Task)
    # Execute and get Task instances; scalars() returns a generator of single-row results.
    tasks = list(session.scalars(stmt))
```

**Select by id:**

```python
with Session(engine) as session:
    # Look up by primary key; returns Task or None if not found.
    task = session.get(Task, task_id)
```

**Insert:**

```python
with Session(engine) as session:
    # Create a Task instance; id and created_at will be set by the database.
    new_task = Task(
        title="Learn SQLAlchemy",
        completed=False,
        priority="high",
    )
    session.add(new_task)       # Mark for insert.
    session.commit()           # Execute INSERT; database assigns id, created_at.
    session.refresh(new_task)  # Reload from DB so new_task.id and new_task.created_at are populated.
```

(With `default=func.now()` on `created_at`, the database sets it automatically.)

**Update:**

```python
with Session(engine) as session:
    task = session.get(Task, task_id)
    if task:
        # Modify attributes; the session tracks changes.
        task.title = "Updated title"
        session.commit()  # Execute UPDATE; database sets updated_at via onupdate.
```

(With `onupdate=func.now()` on `updated_at`, the database sets it when the row is updated.)

**Delete:**

```python
with Session(engine) as session:
    task = session.get(Task, task_id)
    if task:
        session.delete(task)  # Mark for deletion.
        session.commit()     # Execute DELETE.
```

---

## FastAPI Integration: Session Per Request

In a web app, each HTTP request should use its own session and close it when the request ends. FastAPI’s **dependency injection** works well for this:

```python
from sqlalchemy.orm import Session, sessionmaker

# Factory for creating sessions; one session per request.
# autocommit=False: explicit commit required. autoflush=False: no auto-flush before queries.
SessionLocal = sessionmaker(autocommit=False, autoflush=False, bind=engine)

def get_db():
    db = SessionLocal()
    try:
        yield db   # Provide session to the route handler.
    finally:
        db.close()  # Always close when the request ends, even if an error occurred.
```

Register `get_db` as a dependency and inject it into route handlers:

```python
from fastapi import Depends

@app.get("/api/tasks")
def list_tasks(db: Session = Depends(get_db)):
    # FastAPI injects the session from get_db; it's closed automatically after the request.
    stmt = select(Task)
    tasks = list(db.scalars(stmt))
    return tasks  # FastAPI serializes the result to JSON.
```

Your **repository** would receive `db` (or get it via a dependency) and use it for queries. The service layer calls the repository; it does not create sessions itself.

---

## Repository Pattern with SQLAlchemy

Replace the in-memory store with a repository that uses the database:

```python
# repositories/task_repository.py — Data access layer; uses Session for all DB operations.
from sqlalchemy import select
from sqlalchemy.orm import Session

from models import Task  # SQLAlchemy model

def get_all_tasks(db: Session) -> list[Task]:
    stmt = select(Task)
    return list(db.scalars(stmt))

def get_task_by_id(db: Session, task_id: int) -> Task | None:
    return db.get(Task, task_id)

def create_task(db: Session, title: str, completed: bool = False, priority: str = "medium") -> Task:
    task = Task(title=title, completed=completed, priority=priority)
    db.add(task)
    db.commit()
    db.refresh(task)  # Load id, created_at from database.
    return task

def update_task(db: Session, task_id: int, **updates) -> Task | None:
    task = db.get(Task, task_id)
    if not task:
        return None
    # Apply only provided updates.
    for key, value in updates.items():
        if hasattr(task, key):
            setattr(task, key, value)
    db.commit()
    db.refresh(task)  # Load updated_at from database.
    return task

def delete_task(db: Session, task_id: int) -> bool:
    task = db.get(Task, task_id)
    if not task:
        return False
    db.delete(task)
    db.commit()
    return True
```

The **service** layer would receive `db` via dependency injection and pass it to the repository. The **routes** stay the same — they call the service, which calls the repository.

---

## ORM Models vs Pydantic Models

You now have two kinds of models:

| Purpose      | Type       | Used in                      |
| ------------ | ---------- | ---------------------------- |
| **Database** | SQLAlchemy | Repository, table definition |
| **API**      | Pydantic   | Request/response, validation |

The repository returns SQLAlchemy models. Before sending to the client, convert them to Pydantic (or dicts). Example:

```python
# Pydantic model for API response; used for request/response validation and JSON serialization.
# datetime fields serialize to ISO 8601 strings in JSON (e.g. "2026-02-09T12:00:00").
from datetime import datetime

class TaskResponse(BaseModel):
    id: int
    title: str
    completed: bool
    priority: str
    createdAt: datetime   # Snake_case in DB becomes camelCase in API (convention).
    updatedAt: datetime | None

# Convert SQLAlchemy model (database) to Pydantic model (API).
# The repository returns Task; convert before returning from the route.
def task_to_response(task: Task) -> TaskResponse:
    return TaskResponse(
        id=task.id,
        title=task.title,
        completed=task.completed,
        priority=task.priority,
        createdAt=task.created_at,   # Map snake_case to camelCase.
        updatedAt=task.updated_at,
    )
```

---

## Async Option (Brief)

For high concurrency, SQLAlchemy 2 supports **async** with `create_async_engine` and `AsyncSession`:

```python
from sqlalchemy.ext.asyncio import create_async_engine, AsyncSession, async_sessionmaker

# Use asyncpg driver; connection string must use postgresql+asyncpg://.
engine = create_async_engine("postgresql+asyncpg://user:password@localhost:5432/db")
# expire_on_commit=False: objects remain usable after commit (needed for async).
AsyncSessionLocal = async_sessionmaker(engine, class_=AsyncSession, expire_on_commit=False)

async def get_db():
    # Async context manager; use with Depends(get_db) in async route handlers.
    async with AsyncSessionLocal() as session:
        yield session
```

Use `await session.execute(stmt)` and `await session.commit()`. FastAPI route handlers must be `async def`. For many apps, sync SQLAlchemy is simpler and sufficient.

---

## Hands-On: Update Your Task Tracker to Use PostgreSQL

See **Exercise 4** for step-by-step instructions to upgrade your Task Tracker backend from Exercise 3 to use PostgreSQL, SQLAlchemy 2, and Alembic. You will:

- Replace the in-memory store with PostgreSQL
- Use numeric IDs and datetime for `createdAt` / `updatedAt`
- Add a repository layer backed by SQLAlchemy sessions
- Use Alembic for schema migrations instead of `create_all()`
- Update the frontend `Task` type to match the new API

---

## Summary: Key Concepts

| Concept          | Meaning                                                                               |
| ---------------- | ------------------------------------------------------------------------------------- |
| **PostgreSQL**   | Relational database; backend connects via connection string                           |
| **ORM**          | Map Python classes to tables; SQLAlchemy generates SQL                                |
| **Engine**       | Manages connections; create once at startup                                           |
| **Session**      | Per-request “conversation” with DB; use context manager or dependency                 |
| **Model**        | Python class with `Mapped` / `mapped_column`; maps to a table                         |
| `**select()`\*\* | Build SELECT statements; `session.scalars(stmt)` returns objects                      |
| **Repository**   | Data access layer; uses Session to run queries, returns domain objects                |
| **Alembic**      | Migration tool; versioned schema changes, autogenerate from models, upgrade/downgrade |

---

## What You Should Understand After This Topic

By the end, you should be able to:

- Explain what PostgreSQL is and why it is used for persistent storage.
- Describe what an ORM does and why SQLAlchemy is used.
- Define SQLAlchemy 2 models with `DeclarativeBase`, `Mapped`, and `mapped_column`.
- Create an engine and session, and use `select()`, `session.add()`, `session.commit()`, `session.delete()` for CRUD.
- Integrate SQLAlchemy with FastAPI using a `get_db` dependency for session-per-request.
- Place the database in the repository layer; services call the repository, not the database directly.
- Distinguish SQLAlchemy models (database) from Pydantic models (API).
- Set up Alembic, generate migrations with `alembic revision --autogenerate`, and apply them with `alembic upgrade head`.

**Exercise 4** guides you through replacing the in-memory store with PostgreSQL and SQLAlchemy in your Task Tracker backend (Exercise 3), using Alembic for schema migrations.
