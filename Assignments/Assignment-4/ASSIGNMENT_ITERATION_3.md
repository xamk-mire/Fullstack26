# Assignment: Iteration 3 — PostgreSQL Database and Exercise Create/Update

**Project:** Exercise/Workout Tracker  
**Iteration:** 3 of 4  
**Focus:** Replace in-memory storage with PostgreSQL, use an ORM, then add the ability to create new exercises and update existing ones.

**Prerequisite:** Completed [ASSIGNMENT_ITERATION_2.md](./Assignments/Assignment-3/ASSIGNMENT_ITERATION_2.md) (FastAPI Backend and Frontend Integration). You will use the existing backend and frontend and extend them with database persistence and create/update functionality.

---

## Introduction

In Iteration 2 you built a FastAPI backend with layered architecture (routes → services → repository) and connected the frontend to it. The repository stored data **in memory**, so it was lost when the server restarted. In this iteration you will:

1. **Add PostgreSQL** as the persistent data store. You will install PostgreSQL (locally or via Docker), create a database, and use **SQLAlchemy 2** to define tables, perform queries, and handle connections.
2. **Replace the in-memory repository** with a database-backed repository. The service layer and routes stay largely the same; only the repository implementation changes. Use SQL scripts to create the schema and **seed** initial data (categories, exercises, exercise–category links, exercise fields) so the app behaves as before.
3. **Add create and update for exercises.** Once the backend persists to the database, you will add `POST /api/exercises` (create) and `PUT /api/exercises/{id}` (update). The frontend will get new pages/forms: **Create exercise** and **Edit exercise**, wired to these endpoints.

By the end, the Exercise Tracker will persist data in PostgreSQL and support creating and editing exercises. Categories remain read-only for now; you can extend them later.

---

## Overall goal

1. **PostgreSQL database:**

- Install and run PostgreSQL (locally or via Docker).
- Create a database (e.g. `exercise_tracker`) for the application.
- Define schema: `categories`, `exercises`, `exercise_categories` (many-to-many link), `exercise_fields`. Use **integer primary keys** (SERIAL) to match the existing API contract (frontend expects integer ids).

2. **ORM (SQLAlchemy 2):**

- Install **SQLAlchemy 2.x** and a PostgreSQL driver (e.g. `psycopg2-binary` for sync, or `asyncpg` for async).
- Define **ORM models** using the SQLAlchemy 2 declarative style (`DeclarativeBase`, `Mapped`, `mapped_column`). Keep the existing Pydantic DTOs in `models/dtoModels.py` for API request/response; the ORM models are for database access.
- Replace `repositories/store.py` with a database-backed repository that uses SQLAlchemy 2’s `select()`, `session.execute()`, `session.add()`, etc. The service layer continues to call the same logical operations; only the implementation changes from in-memory lists to database queries.
- Use **migrations** (e.g. Alembic) or SQL scripts to create tables and optionally seed initial data.

3. **Configuration:**

- Database URL from an environment variable (e.g. `DATABASE_URL` or `POSTGRES_URL`). The backend reads this at startup and passes it to the ORM/session factory.

4. **Create and update exercises:**

- **Backend:** Add `POST /api/exercises` (create) and `PUT /api/exercises/{id}` (update). Request bodies will include `name`, `notes`, `date`, `categoryIds` (list of category ids), and `fields` (list of { name, value, unit }). The service creates or updates the exercise and its categories and fields in the database; the repository exposes create and update functions.
- **Frontend:** Add a **Create exercise** form (e.g. at `/exercises/new`) and an **Edit exercise** form (e.g. at `/exercises/:id/edit`). Both forms collect name, notes, date, category selection, and fields. Wire them to the new API endpoints via the api service and API client. After a successful create, redirect to the new exercise’s detail page; after update, redirect to the detail page or list.

---

## What you need before starting

- **Iteration 2 solution** (backend with in-memory store, frontend with api service and API client). The backend has `repositories/store.py`, `services/exercise_service.py`, `services/category_service.py`, and routes for categories and exercises. The frontend has `api/exercisesApi.ts`, `apiService.ts`, and pages for the exercise list and detail.
- **PostgreSQL** installed locally or via Docker. You will create a database and run migrations or SQL scripts.
- **Python 3.10+** and the backend virtual environment from Iteration 2.
- Basic familiarity with **SQL** and **ORMs** from course materials. You will use **SQLAlchemy 2** (e.g. `sqlalchemy>=2.0`).

---

## Part A — PostgreSQL and ORM integration

### A.1 — Install and run PostgreSQL

1. **Install PostgreSQL** on your machine or use **Docker**. If using Docker, you can run a PostgreSQL container with a published port (e.g. 5432) and a volume for data persistence. Create a database (e.g. `exercise_tracker`) using the PostgreSQL client (`psql`) or a GUI tool.
2. **Note the connection details:** host (usually `localhost`), port (default `5432`), database name, username, and password. You will use these to build the database URL (e.g. `postgresql://user:password@localhost:5432/exercise_tracker`).

### A.2 — Add database dependencies and configuration

1. **Install SQLAlchemy 2** and a PostgreSQL driver in the backend. Use `sqlalchemy>=2.0` (e.g. `^2.0` or `^2.1`) and `psycopg2-binary` for sync, or `asyncpg` if you prefer async. Add **Alembic** (`alembic`) if you want versioned migrations, or prepare SQL scripts for creating and seeding the schema.
2. **Database URL from environment:** Add support for a `DATABASE_URL` (or similar) environment variable. The backend should read this at startup; if it is not set, you can default to a local development URL or fail fast with a clear error. Do not hardcode passwords.
3. **Database session or engine:** Create a module (e.g. `backend/database.py` or `backend/db/session.py`) that creates a SQLAlchemy 2 engine and `sessionmaker` (or `async_sessionmaker` for async) using the database URL. The repository will use this to obtain sessions for queries and writes.

### A.3 — Define ORM models and schema (SQLAlchemy 2 style)

1. **ORM models:** Use the **SQLAlchemy 2 declarative style**. Create a base class with `DeclarativeBase`, then define models using `Mapped` and `mapped_column`:

- **Category:** `id: Mapped[int]` (primary key, auto-increment), `name: Mapped[str]`, `description: Mapped[str]`.
- **Exercise:** `id: Mapped[int]` (primary key, auto-increment), `name: Mapped[str]`, `notes: Mapped[str | None]`, `date: Mapped[datetime]` (or `str` if you store as text).
- **ExerciseCategory:** association table or model for the many-to-many link. Columns: `exercise_id` (FK), `category_id` (FK). Use a composite primary key or a separate `id`.
- **ExerciseField:** `id: Mapped[int]`, `exercise_id: Mapped[int]` (FK), `name: Mapped[str]`, `value: Mapped[str]`, `unit: Mapped[str]`.

2. **Relationships:** Use `relationship()` with the SQLAlchemy 2 style (e.g. `back_populates`, `secondary` for many-to-many) so you can load an exercise with its categories and fields (e.g. `exercise.categories`, `exercise.fields`). Use `selectinload` or `joinedload` for eager loading in queries.
3. **Match existing data shape:** The current in-memory store uses integer ids and stores `date` as an ISO string. Keep the API response format the same (integer ids, date as string) so the frontend does not need to change for Part A. Convert `datetime` to ISO string in the service or repository when building DTOs.

### A.3.1 — SQLAlchemy 2 query style

Use the **SQLAlchemy 2 statement API** in the repository:

- **Select:** `select(Model).where(...)` and `session.execute(select(...))` instead of `session.query(Model)`.
- **Get by primary key:** `session.get(Model, id)`.
- **Insert:** `session.add(instance)` and `session.commit()`, or `session.add_all([...])`.
- **Update:** modify attributes on the instance and `session.commit()`, or use `update(Model).where(...).values(...)` with `session.execute()`.
- **Delete:** `session.delete(instance)` or `delete(Model).where(...)` with `session.execute()`.

### A.4 — Create database schema (migrations or SQL)

1. **Migrations (Alembic):** If you use Alembic, initialize it in the backend, configure it to use your models and database URL, and generate an initial migration that creates the four tables (`categories`, `exercises`, `exercise_categories`, `exercise_fields`). Run the migration to apply the schema.
2. **Or SQL scripts:** Alternatively, write a SQL script that creates the tables with the correct columns and constraints. Run it against your database.

### A.4.1 — Seeding examples

Seed data must match the Iteration 2 in-memory store so the app behaves the same. Insert in this order: categories, exercises, exercise_categories, exercise_fields (to satisfy foreign keys).

**Option 1 — Raw SQL**

Run this against your database (adjust table names if they differ, e.g. snake_case):

```sql
-- Categories (ids 1–5)
INSERT INTO categories (id, name, description) VALUES
  (1, 'Cardio', 'Cardiovascular exercises'),
  (2, 'Strength', 'Resistance and weight training'),
  (3, 'Running', 'Running and jogging'),
  (4, 'Swimming', 'Pool and open water swimming'),
  (5, 'Trail', 'Trail and outdoor activities');

-- Exercises (ids 1–6)
INSERT INTO exercises (id, name, notes, date) VALUES
  (1, 'Bench press', 'Flat bench, felt strong', '2025-02-01 10:00:00'),
  (2, 'Morning run', 'Easy 5k along the river', '2025-02-02 07:30:00'),
  (3, 'Swimming laps', '30 min freestyle', '2025-02-02 18:00:00'),
  (4, 'Squats', 'Back squat 4x5', '2025-02-03 09:00:00'),
  (5, 'Trail run', 'Hilly 8k', '2025-01-28 08:00:00'),
  (6, 'Deadlift', 'Conventional 3x5', '2025-02-01 10:45:00');

-- Exercise–category links (exerciseId, categoryId)
INSERT INTO exercise_categories (exercise_id, category_id) VALUES
  (1, 2), (4, 2), (6, 2),
  (2, 1), (2, 3), (3, 1), (3, 4), (5, 1), (5, 3), (5, 5);

-- Exercise fields (exerciseId, name, value, unit)
INSERT INTO exercise_fields (exercise_id, name, value, unit) VALUES
  (1, 'weight', '100', 'kg'), (1, 'sets', '3', 'reps'), (1, 'reps', '8', 'reps'),
  (2, 'distance', '5', 'km'), (2, 'duration', '30', 'min'),
  (3, 'duration', '30', 'min'), (3, 'laps', '20', 'laps'),
  (4, 'weight', '120', 'kg'), (4, 'sets', '4', 'reps'), (4, 'reps', '5', 'reps'),
  (5, 'distance', '8', 'km'), (5, 'duration', '45', 'min'),
  (6, 'weight', '140', 'kg'), (6, 'sets', '3', 'reps'), (6, 'reps', '5', 'reps');
```

If your tables use `SERIAL` for `id`, omit the `id` column in `categories` and `exercises` and let PostgreSQL generate ids. Then use the generated ids in `exercise_categories` and `exercise_fields`, or run a separate script that queries the inserted rows for their ids. The example above assumes you either use explicit ids or have reset sequences to match.

**Option 2 — Python seed script with SQLAlchemy 2**

Create a script (e.g. `backend/scripts/seed.py` or `backend/seed_db.py`) that uses your ORM models and session:

```python
from datetime import datetime
from sqlalchemy import select
from database import SessionLocal  # or your session factory
from db.models import Category, Exercise, ExerciseCategory, ExerciseField

def seed():
    with SessionLocal() as session:
        # Skip if already seeded
        if session.scalars(select(Category)).first():
            return

        categories = [
            Category(name="Cardio", description="Cardiovascular exercises"),
            Category(name="Strength", description="Resistance and weight training"),
            Category(name="Running", description="Running and jogging"),
            Category(name="Swimming", description="Pool and open water swimming"),
            Category(name="Trail", description="Trail and outdoor activities"),
        ]
        session.add_all(categories)
        session.flush()  # Get generated ids

        exercises = [
            Exercise(name="Bench press", notes="Flat bench, felt strong", date=datetime(2025, 2, 1, 10, 0, 0)),
            Exercise(name="Morning run", notes="Easy 5k along the river", date=datetime(2025, 2, 2, 7, 30, 0)),
            Exercise(name="Swimming laps", notes="30 min freestyle", date=datetime(2025, 2, 2, 18, 0, 0)),
            Exercise(name="Squats", notes="Back squat 4x5", date=datetime(2025, 2, 3, 9, 0, 0)),
            Exercise(name="Trail run", notes="Hilly 8k", date=datetime(2025, 1, 28, 8, 0, 0)),
            Exercise(name="Deadlift", notes="Conventional 3x5", date=datetime(2025, 2, 1, 10, 45, 0)),
        ]
        session.add_all(exercises)
        session.flush()

        # exercise_categories: (exercise_id, category_id) — Bench(1)→Strength(2), Squats(4)→Strength(2), etc.
        links = [(1, 2), (4, 2), (6, 2), (2, 1), (2, 3), (3, 1), (3, 4), (5, 1), (5, 3), (5, 5)]
        for ex_id, cat_id in links:
            session.add(ExerciseCategory(exercise_id=ex_id, category_id=cat_id))

        # exercise_fields: (exercise_id, name, value, unit)
        fields_data = [
            (1, "weight", "100", "kg"), (1, "sets", "3", "reps"), (1, "reps", "8", "reps"),
            (2, "distance", "5", "km"), (2, "duration", "30", "min"),
            (3, "duration", "30", "min"), (3, "laps", "20", "laps"),
            (4, "weight", "120", "kg"), (4, "sets", "4", "reps"), (4, "reps", "5", "reps"),
            (5, "distance", "8", "km"), (5, "duration", "45", "min"),
            (6, "weight", "140", "kg"), (6, "sets", "3", "reps"), (6, "reps", "5", "reps"),
        ]
        for ex_id, name, value, unit in fields_data:
            session.add(ExerciseField(exercise_id=ex_id, name=name, value=value, unit=unit))

        session.commit()
```

Run the script with `python -m scripts.seed` or `python seed_db.py` from the backend directory. Adjust imports and model names to match your project. The `links` and `fields_data` tuples use 1-based exercise and category ids assuming they are generated in insertion order; if your ORM assigns different ids, query the created `Exercise` and `Category` instances for their ids and build the links accordingly.

### A.5 — Replace the in-memory repository with database-backed implementation

1. **New repository implementation:** Create a new repository module (or replace the logic in `store.py`) that uses **SQLAlchemy 2** (e.g. `select()`, `session.execute()`, `session.get()`). Implement the same functions the service layer expects:

- `get_all_categories()` → query the Category table, return list of records (id, name, description).
- `get_category_by_id(id)` → query by id, return one or None.
- `get_all_exercises()` → query Exercise, optionally eager-load categories and map to records.
- `get_exercise_by_id(id)` → query by id, return one or None, with categories and fields.
- `get_exercise_category_ids(exercise_id)` → query the link table, return list of category ids.
- `get_exercise_fields(exercise_id)` → query ExerciseField by exercise_id, return list of records.

2. **Use sessions:** Each function should obtain a database session (from your session factory or FastAPI dependency), execute the query, and return the data. Handle sessions in a consistent way (e.g. context manager or dependency injection) so connections are properly closed.
3. **Update the backend to use the new repository:** Ensure the service layer imports and uses the database-backed repository instead of the old in-memory store. The route and service code should require minimal changes—only the source of data changes.
4. **Seed data:** If you have not already, run a seed script or migration that inserts the same categories, exercises, exercise–category links, and exercise fields as in the Iteration 2 store. This lets you verify that the app still works: list and detail pages should show the same data, now from the database.

### A.6 — Verify Part A

- Start the backend with the `DATABASE_URL` environment variable set. The backend should connect to PostgreSQL and start without errors.
- Open the frontend and confirm the exercise list and exercise detail pages load data from the database. The behaviour should match Iteration 2; only the storage has changed.
- Restart the backend and reload the frontend; data should persist (not reset to a default state).

---

## Part B — Create and update exercises

### B.1 — Backend: request DTOs and repository create/update

1. **Request DTOs:** Add Pydantic models for create and update in `models/dtoModels.py` (or a separate file):

- **ExerciseCreateRequest** (or similar): `name` (str), `notes` (str, optional), `date` (str, ISO format), `categoryIds` (list[int]), `fields` (list of objects with `name`, `value`, `unit`). All required except `notes` if you allow empty notes.
- **ExerciseUpdateRequest:** same fields as create; all optional so the client can send a partial update, or require all fields—your choice. For simplicity, you can use the same shape as create and require all fields for update.

2. **Repository create:** Implement a function such as `create_exercise(name, notes, date, category_ids, fields)` in the repository. It should:

- Insert a new row into the `exercises` table and obtain the generated `id`.
- Insert rows into `exercise_categories` for each category id.
- Insert rows into `exercise_fields` for each field (with the new exercise id).
- Use a transaction so that if any step fails, the whole operation is rolled back.
- Return the new exercise record (or id) so the service can build the full DTO.

3. **Repository update:** Implement a function such as `update_exercise(exercise_id, name, notes, date, category_ids, fields)`. It should:

- Update the exercise row (name, notes, date).
- Replace the exercise’s categories: delete existing links for this exercise, insert new ones for the given `category_ids`.
- Replace the exercise’s fields: delete existing fields for this exercise, insert new ones from the `fields` list.
- Use a transaction. Return the updated exercise record or None if the exercise does not exist.

### B.2 — Backend: service layer create and update

1. **Create exercise:** In `exercise_service.py`, implement a function (e.g. `create_exercise(data: ExerciseCreateRequest)`) that:

- Calls the repository’s create function with the data from the request.
- Builds an `ExerciseDetailDto` from the created exercise (with categories and fields) and returns it. The service is responsible for the DTO shape; the repository returns raw records.

2. **Update exercise:** Implement a function (e.g. `update_exercise(exercise_id: int, data: ExerciseUpdateRequest)`) that:

- Calls the repository’s update function.
- If the exercise does not exist, return None.
- Otherwise build and return an `ExerciseDetailDto` for the updated exercise.

### B.3 — Backend: routes for create and update

1. **POST /api/exercises:** In `routes/exercises.py`, add a POST route. It should:

- Accept a JSON body that matches `ExerciseCreateRequest`.
- Call the exercise service’s create function.
- Return the created `ExerciseDetailDto` with status 201 (Created). Set the `Location` header to the new exercise’s URL (e.g. `/api/exercises/{id}`) if desired.
- Validate the request body (Pydantic will do this if you use the request model). Return 422 for validation errors.

2. **PUT /api/exercises/{id}:** Add a PUT route. It should:

- Accept the path parameter `id` (int) and a JSON body matching `ExerciseUpdateRequest`.
- Call the exercise service’s update function.
- If the exercise does not exist, return 404.
- Otherwise return the updated `ExerciseDetailDto` with status 200.
- Return 422 for validation errors.

### B.4 — Frontend: API client and api service

1. **API client:** In `api/exercisesApi.ts`, add:

- `createExercise(data: ExerciseCreateRequest): Promise<ExerciseDetailDto>` — POST to `/api/exercises`, send the body as JSON, return the response.
- `updateExercise(id: number, data: ExerciseUpdateRequest): Promise<ExerciseDetailDto | null>` — PUT to `/api/exercises/{id}`, send the body, return the response or null if 404.

2. **Request types:** In `frontend/src/types/index.ts` (or a dedicated file), add TypeScript interfaces for the create and update request bodies. They should match the backend’s `ExerciseCreateRequest` and `ExerciseUpdateRequest` (e.g. `name`, `notes`, `date`, `categoryIds`, `fields`).
3. **Api service:** In `apiService.ts`, add `createExercise` and `updateExercise` that delegate to the API client. Map `null` to `undefined` if your pages expect `undefined` for “not found”.

### B.5 — Frontend: Create exercise form

1. **Route:** Add a route for the create form (e.g. `/exercises/new`). Use a new page component (e.g. `ExerciseFormPage` or `CreateExercisePage`).
2. **Form fields:** The form should collect:

- **Name** (required): text input.
- **Notes** (optional): textarea.
- **Date** (required): date-time input (or date + time). Use a value that serializes to ISO format (e.g. `2025-02-03T10:00:00`) for the API.
- **Categories:** multi-select or checkboxes for the list of categories (from `getCategories()`). The user selects which categories apply; the form submits `categoryIds` as an array of numbers.
- **Fields:** a list of exercise fields. Each field has `name` (e.g. "weight"), `value` (e.g. "100"), `unit` (e.g. "kg"). Allow the user to add and remove fields dynamically. The form submits `fields` as an array of `{ name, value, unit }`.

3. **Submit:** On submit, call `createExercise` with the form data. On success, redirect to the new exercise’s detail page (e.g. `/exercises/{id}`). On error (e.g. validation or network), show an error message.
4. **Navigation:** Add a link or button on the exercise list page (e.g. “Add exercise” or “New exercise”) that navigates to `/exercises/new`.

### B.6 — Frontend: Edit exercise form

1. **Route:** Add a route for the edit form (e.g. `/exercises/:id/edit`). Use a page component that loads the existing exercise by id and pre-fills the form.
2. **Load existing data:** When the page mounts, call `getExerciseById(id)`. If the exercise is not found (undefined), show a “Not found” message and a link back. Otherwise, populate the form with the exercise’s name, notes, date, selected categories, and fields.
3. **Form:** Reuse the same form structure as the create form (or a shared component). The form fields and validation are the same; only the initial values and the submit action differ.
4. **Submit:** On submit, call `updateExercise(id, data)` with the form data. On success, redirect to the exercise detail page. On error, show an error message.
5. **Navigation:** Add an “Edit” button on the exercise detail page that navigates to `/exercises/{id}/edit`.

### B.7 — Verify Part B

- Create a new exercise from the form. Confirm it appears in the list and detail view and is stored in the database (survives a backend restart).
- Edit an existing exercise. Confirm the changes persist and are reflected in the list and detail views.
- Verify that validation errors (e.g. missing name) are shown and that invalid ids (e.g. edit for non-existent exercise) are handled gracefully.

---

## Summary

After completing this assignment you will have:

- **PostgreSQL** as the persistent store, with tables for categories, exercises, exercise_categories, and exercise_fields.
- **ORM (SQLAlchemy 2)** for database access. The repository uses SQLAlchemy 2’s declarative models and statement API (`select()`, `session.execute()`, etc.) instead of in-memory lists; the service and route layers are unchanged in structure.
- **Database configuration** via environment variable. Migrations or SQL scripts create and optionally seed the schema.
- **Create and update exercises:** `POST /api/exercises` and `PUT /api/exercises/{id}` in the backend; create and edit forms in the frontend, wired to the api service and API client.
- The exercise list and detail pages continue to work; data now persists across restarts, and users can add and edit exercises.

---

## Reporting

Follow the instructions in **[ASSIGMENT_ITERATION_3_REPORTING.md](./Assignments/Assignment-4/ASSIGNMENT_ITERATION_3_REPORTING.md)

---

_End of Assignment — Iteration 3_
